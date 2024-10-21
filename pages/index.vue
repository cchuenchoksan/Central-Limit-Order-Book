<template>
  <div>
    <input
      hidden
      id="uploader"
      type="file"
      @change="handleFileUpload"
      accept=".csv"
    />
    <button
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded m-4"
    >
      <label for="uploader" style="cursor: pointer">Upload data</label>
    </button>
    <button
      @click="onClick()"
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded m-4"
    >
      Next order
    </button>
    <div class="border rounded p-4 m-4">
      <div class="grid grid-cols-6 gap-4">
        <!-- ROW 1 -->
        <div class="attribute-name">Name:</div>
        <div class="attribute-val">GemTrade</div>

        <div class="attribute-name">Volume:</div>
        <div class="attribute-val">{{ volume }}</div>

        <div class="attribute-name">Open (first trade):</div>
        <div v-if="tradeLog[0] !== undefined" class="attribute-val">
          {{ tradeLog[0].price }}
        </div>
        <div v-else class="attribute-val">No trade</div>

        <!-- ROW 2 -->
        <div class="attribute-name">Last:</div>
        <div class="attribute-val">{{ last }}</div>

        <div class="attribute-name">Bid-Ask Spread (ticks):</div>
        <div v-if="rowsData[0] !== undefined" class="attribute-val">
          {{
            (
              (rowsData[0].askPriceRow - rowsData[0].bidPriceRow) /
              0.01
            ).toFixed(0)
          }}
        </div>
        <div v-else class="attribute-val"></div>

        <div class="attribute-name">High:</div>
        <div class="attribute-val">{{ high }}</div>

        <!-- ROW 3 -->
        <div class="attribute-name">Change (from open):</div>
        <div v-if="tradeLog[0] !== undefined" class="attribute-val">
          {{
            (((last - tradeLog[0].price) * 100) / tradeLog[0].price).toFixed(2)
          }}%
        </div>
        <div v-else class="attribute-val">No trade</div>

        <div class="attribute-name">Average best depth (USD):</div>
        <div v-if="rowsData[0] !== undefined" class="attribute-val">
          {{
            (
              (rowsData[0].askPriceRow * rowsData[0].askVolRow +
                rowsData[0].bidPriceRow * rowsData[0].bidVolRow) /
              2
            ).toFixed(2)
          }}
        </div>
        <div v-else class="attribute-val"></div>

        <div class="attribute-name">Low:</div>
        <div v-if="low !== 9999" class="attribute-val">{{ low }}</div>
        <div v-else class="attribute-val">0</div>
      </div>

      <div class="grid grid-cols-12 gap-2 mt-3" v-for="row in rowsData">
        <clobRow
          :bid-vol="row.bidVolRow"
          :bid-price="row.bidPriceRow"
          :ask-price="row.askPriceRow"
          :ask-vol="row.askVolRow"
          :max-vol="this.maxVol"
        />
      </div>
    </div>
    <div class="border rounded p-4 m-4">
      <h1 class="my-2" style="font-size: xx-large; font-weight: bold">
        Trade Log
      </h1>
      <div class="grid grid-cols-5 mb-2">
        <div style="font-weight: bold">Timestamp</div>
        <div style="font-weight: bold">Sell Group ID</div>
        <div style="font-weight: bold">Buy Group ID</div>
        <div style="font-weight: bold">Price</div>
        <div style="font-weight: bold">Quantity</div>
      </div>
      <div class="grid grid-cols-5" v-for="trade in tradeLog">
        <div>{{ new Date(trade.timestamp).toISOString() }}</div>
        <div>{{ trade.sellOrderId }}</div>
        <div>{{ trade.buyOrderId }}</div>
        <div>{{ trade.price }}</div>
        <div>{{ trade.quantity }}</div>
      </div>
    </div>
    <div class="border rounded p-4 m-4">
      <h1 class="my-2" style="font-size: xx-large; font-weight: bold">
        Pending Orders
      </h1>
      <div class="grid grid-cols-5 mb-2">
        <div style="font-weight: bold">Timestamp</div>
        <div style="font-weight: bold">Side</div>
        <div style="font-weight: bold">Group ID</div>
        <div style="font-weight: bold">Price</div>
        <div style="font-weight: bold">Quantity</div>
      </div>
      <div class="grid grid-cols-5" v-for="order in initOrders">
        <div>{{ new Date(order.timestamp).toISOString() }}</div>
        <div>{{ order.side }}</div>
        <div>{{ order.groupId }}</div>
        <div>{{ order.price }}</div>
        <div>{{ order.size }}</div>
      </div>
    </div>
  </div>
</template>
<script>
import Papa from "papaparse";

export default {
  data() {
    return {
      initOrders: [],
      buyOrders: [],
      sellOrders: [],
      tradeLog: [],
      rowsData: [],
      maxVol: 0,
      volume: 0,
      last: 0,
      high: 0,
      low: 9999, // NOTE: this assumes the starting price does not exceed 9999
    };
  },
  methods: {
    onClick() {
      if (this.initOrders.length === 0) {
        alert("No pending order to process!");
      }
      var current_order = this.initOrders.shift();
      this.addOrder(current_order);
      this.matchOrder();
      this.createRows();
    },

    sortOrders(array, ascending) {
      var direction;
      if (ascending === true) {
        direction = 1;
      } else {
        direction = -1;
      }
      return array.sort((a, b) => {
        if (a.price === b.price) {
          return direction * (a.timestamp - b.timestamp);
        } else {
          return direction * (a.price - b.price);
        }
      });
    },

    addOrder(order) {
      order["timestamp"] = new Date(order["timestamp"]);
      if (order.side === "buy") {
        this.buyOrders.push(order);
        this.buyOrders = this.sortOrders(this.buyOrders, false);
      } else {
        this.sellOrders.push(order);
        this.sellOrders = this.sortOrders(this.sellOrders, true);
      }
    },

    matchOrder() {
      while (
        this.buyOrders.length !== 0 &&
        this.sellOrders.length !== 0 &&
        this.buyOrders[0].price >= this.sellOrders[0].price
      ) {
        var buyOrder = this.buyOrders[0];
        var sellOrder = this.sellOrders[0];

        var matchQuantity = Math.min(buyOrder.size, sellOrder.size);

        var matchPrice;
        if (buyOrder.timestamp > sellOrder.timestamp) {
          matchPrice = sellOrder.price;
        } else {
          matchPrice = buyOrder.price;
        }
        var tradeTimestamp = Math.max(buyOrder.timestamp, sellOrder.timestamp);
        this.tradeLog.push({
          buyOrderId: buyOrder.groupId,
          sellOrderId: sellOrder.groupId,
          price: matchPrice,
          quantity: matchQuantity,
          timestamp: tradeTimestamp,
        });

        this.volume += matchQuantity;
        this.last = matchPrice;
        this.high = Math.max(this.high, matchPrice);
        this.low = Math.min(this.low, matchPrice);

        this.buyOrders[0].size -= matchQuantity;
        this.sellOrders[0].size -= matchQuantity;

        if (this.buyOrders[0].size === 0) {
          this.buyOrders.shift();
        }
        if (this.sellOrders[0].size === 0) {
          this.sellOrders.shift();
        }
      }
    },

    createCLOB() {
      var bidData = {};
      for (var i = 0; i < this.buyOrders.length; i++) {
        var priceString = this.buyOrders[i].price.toFixed(2);
        if (priceString in bidData) {
          bidData[priceString] += this.buyOrders[i].size;
        } else {
          bidData[priceString] = this.buyOrders[i].size;
        }
      }

      var askData = {};
      for (var i = 0; i < this.sellOrders.length; i++) {
        var priceString = this.sellOrders[i].price.toFixed(2);
        if (priceString in askData) {
          askData[priceString] += this.sellOrders[i].size;
        } else {
          askData[priceString] = this.sellOrders[i].size;
        }
      }

      var bidDataArray = Object.entries(bidData);
      var askDataArray = Object.entries(askData);

      bidDataArray.sort((a, b) => {
        if (parseFloat(a[0]) < parseFloat(b[0])) return 1;
        if (parseFloat(a[0]) > parseFloat(b[0])) return -1;
        return 0;
      });

      askDataArray.sort((a, b) => {
        if (parseFloat(a[0]) < parseFloat(b[0])) return -1;
        if (parseFloat(a[0]) > parseFloat(b[0])) return 1;
        return 0;
      });
      return { bid: bidDataArray, ask: askDataArray };
    },

    createRows() {
      this.rowsData = [];
      var data = this.createCLOB();
      var loopMax = Math.max(data.ask.length, data.bid.length);
      for (var i = 0; i < loopMax; i++) {
        var row = {
          bidVolRow: null,
          bidPriceRow: null,
          askVolRow: null,
          askPriceRow: null,
        };

        if (i < data.ask.length) {
          row.askVolRow = data.ask[i][1];
          row.askPriceRow = data.ask[i][0];
          this.maxVol = Math.max(data.ask[i][1], this.maxVol);
        }

        if (i < data.bid.length) {
          row.bidVolRow = data.bid[i][1];
          row.bidPriceRow = data.bid[i][0];
          this.maxVol = Math.max(data.bid[i][1], this.maxVol);
        }

        this.rowsData.push(row);
      }
    },

    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.parseCSV(file);
      }
    },

    parseCSV(file) {
      Papa.parse(file, {
        complete: (results) => {
          const columnMapping = {
            Timestamp: "timestamp",
            "Group ID": "groupId",
            Side: "side",
            Price: "price",
            Size: "size",
          };

          const transformedData = results.data.map((row) => {
            const newRow = {};
            for (const key in row) {
              if (row.hasOwnProperty(key)) {
                const newKey = columnMapping[key];

                if (newKey === "price" || newKey === "size") {
                  newRow[newKey] = Number(row[key]);
                } else {
                  newRow[newKey] = row[key];
                }
              }
            }
            return newRow;
          });

          this.initOrders = transformedData;
        },
        header: true,
      });
    },
  },
};
</script>
<style>
.fullpage {
  height: 100vh;
}

.attribute-name {
  font-weight: bold;
}

.attribute-val {
  color: blue;
}
</style>
