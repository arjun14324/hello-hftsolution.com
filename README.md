<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <!-- <script src="http://d3js.org/d3.v3.js" charset="utf-8"></script> -->

      <script src="http://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
      <script src="http://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore-min.js"></script>
      <script src="http://cdnjs.cloudflare.com/ajax/libs/async/1.2.1/async.min.js"></script>

      <style>
      h2 {
        color: black;
        text-align: center;
      }

      .axis {
        font-family: arial;
        font-size: 0.6em;
      }

      path {
        fill: none;
        stroke: black;
        stroke-width: 2px;
      }

      .tick {
        fill: none;
        stroke: #777;
        stroke-dasharray: 2,2;
      }

      .axisText {
        stroke-dasharray: none;
        stroke: black;
      }

      circle {
        opacity: 0.4;
        stroke: none;
      }

      .bid {
        fill: blue
      }

      .ask {
        fill: red
      }

      .trade {
        fill: green
      }
    </style>

    <script type="text/javascript">

        "use strict";
        // globals
        var orders, trades,
            previousOrders = [],
            now, yDomain,
            price_scale, time_scale, volume_scale;

        var yRange = [0, 60, 120, 180, 240, 300, 360, 420];

        var margin = 75,
            width = 1400 - margin,
            height = _.last(yRange);

        var buyColor = "blue";
        var sellColor = "red";

        var startTime = 1434529325000;  // Wed, 17 Jun 2015 08:22:05 GMT

        var realStartTime = Date.now();
        var redrawInterval = 200;

        // time in milliseconds
        var oneSecond = 1000;
        var oneMinute = oneSecond * 60;
        var oneHour = oneMinute * 60;
        var oneDay = oneHour * 24;
        var oneWeek = oneDay * 7;
        var twoWeeks = oneWeek * 2;

        function initOrders(data)
        {
            var newOrders = [];

            function initOrder(order, side)
            {
                var newOrder = {};
                newOrder.price = order[0];
                newOrder.volume = order[1];
                newOrder.side = side;

                var previousOrder = _.findWhere(previousOrders, {
                    price: newOrder.price,
                    volume: newOrder.volume,
                    side: side});

                // if can't find a previous order with the same price and volume
                if (!previousOrder)
                {
                    // set timestamp of the new order
                    newOrder.timestamp = Date.now();
//                  newOrder.timestamp = startTime;
                }
                else
                {
                    newOrder.timestamp = previousOrder.timestamp;
                }

                newOrders.push(newOrder);
            }

            data.bids.forEach(function(order)
            {
                initOrder(order, "bid");

            });

            data.asks.forEach(function(order)
            {
                initOrder(order, "ask");
            });

            previousOrders = orders;
            orders = newOrders;
        }

        function renderChart()
        {
            d3.select("body")
              .append("h2")
              .text("Test Order Book");

            var svg = d3.select("body")
              .append("svg")
              .attr("width", width + margin)
              .attr("height", 2 * height)
              .append('g')
              .attr("class",'chart');
        }

        function renderTimeAxis(time_scale)
        {
            var time_axis = d3.svg.axis()
                .scale(time_scale)
                .tickSize(width - margin)
                .tickValues(yDomain)
                .orient("right");

            var orderBookAxis = d3.select("svg")
                .append("g")
                .attr("class", "y axis")
                .attr("transform", "translate(" + margin + ",0)")
                .call(time_axis);

            orderBookAxis.selectAll("text")
                .attr("x", -30)
                .attr("class", "axisText");

            var tradeAxis = d3.select("svg")
                .append("g")
                .attr("class", "trade axis")
                .attr("transform", "translate(" + margin + "," + 2 * height + ") scale(1, -1)")
                .call(time_axis);

            tradeAxis.selectAll("text")
                .attr("x", -30)
                .attr("class", "axisText");
        }

        function renderPriceAxis(price_scale)
        {
            var price_axis = d3.svg.axis()
                .scale(price_scale);

            var xAxis = d3.select("svg")
                .append("g")
                .attr("class", "x axis")
                .attr("transform", "translate(0," + height + ")")
                .call(price_axis);

          xAxis.selectAll("text")
                .attr("class", "axisText")
      }

        function setNow()
        {
            //var now = Date.now();
//            now = startTime + Date.now() - realStartTime;
            now = Date.now();

            yDomain = [
                new Date(now - twoWeeks),
                new Date(now - 15 * oneMinute),
                new Date(now - 5 * oneMinute),
                new Date(now - oneMinute),
                new Date(now - 15 * oneSecond),
                new Date(now - 5 * oneSecond),
                new Date(now - oneSecond),
                new Date(now)];
        }

        function circleRadius(area)
        {
            return Math.sqrt(area / Math.PI);
        }

        function setScales(orders, trades)
        {
            var ordersTrades = orders.concat(trades);

            // Find range of prices across both bids and asks
            var price_extent = d3.extent(ordersTrades, function(d) {
                return d.price;
            });

            // Create x-axis scale mapping order price -> pixels
            price_scale = d3.scale.linear()
                //.domain(price_extent)
                .domain([270,330])
                .range([margin, width]);

            // Find range of volumes across both bids and asks
            var volume_extent = d3.extent(ordersTrades, function(d) {
                return circleRadius(d.volume);
            });

            volume_scale = d3.scale.linear()
                .domain(volume_extent)
                .range([1,50]);

            // Create y-axis scale mapping dates -> pixels
            time_scale = d3.time.scale()
                .domain(yDomain)
                .range(yRange);
        }

        function drawOrders(orders)
        {
            // bind orders to circles
            d3.select("svg")
                // select all circles that do not have a css class od trade
                .selectAll("circle:not(.trade)")
                .data(orders, function(d) {
                    // match orders on price and volume
                    return [d.price, d.volume];
                })
                .enter()
                .append("circle");

            // set circle attributes
            d3.selectAll("circle:not(.trade)")
                .transition()
                .duration(redrawInterval)
                .ease("linear")
                .attr("cy", function(d) {
                    return time_scale(d.timestamp);
                })
                .attr("cx", function(d) {
                    return price_scale(d.price);
                })
                .attr("r", function(d) {
                    return volume_scale(circleRadius(d.volume) );
                })
                .attr("class", function(d) {
                  return d.side;
                });
        }

        function drawTrades(trades)
        {
            // bind trades to circles with trade css class
            d3.select("svg")
                .selectAll("circle.trade")
                .data(trades, function(d) {
                    // match trades on price, volume and timestamp
                    return [d.price, d.volume, d.timestamp];
                })
                .enter()
                .append("circle")
                  .attr("class","trade");

            // set trade circle attributes
            d3.selectAll("circle.trade")
                .transition()
                .duration(redrawInterval)
                .ease("linear")
                .attr("cy", function(d) {
                    return 2 * height - time_scale(d.timestamp);
                })
                .attr("cx", function(d) {
                    return price_scale(d.price);
                })
                .attr("r", function(d) {
                    return volume_scale(circleRadius(d.volume) );
                })
        }

        function getOrders(callback)
        {
            d3.json("orderDataTestBTCAUD.json", function(error, json)
            {
                if (error) {
                  console.warn(error);
                  return callback(error);
                }

                initOrders(json);

                callback(null, orders);
            });
        }

        function getTrades(callback)
        {
            d3.json("tradesTestBTCAUD.json", function(error, json)
            {
                if (error) {
                  console.warn(error);
                  return callback(error);
                }

                trades = [];

                json.forEach(function(trade)
                {
                    var newTrade = {};
                    newTrade.price = trade[0];
                    newTrade.volume = trade[1];
                    //TODO remove as only in for testing
                    newTrade.timestamp = trade[2];

                    trades.push(newTrade);
                });

                callback(null, trades);
            });
        }

        function getData(callback)
        {
            async.parallel([getOrders, getTrades],
                function(error, results)
                {
                    if (error) {
                        console.warn(error);
                        return callback(error);
                    }

                    var orders = results[0],
                        trades = results[1];

                    callback(null, orders, trades);
                });
        }

        function redraw(orders, trades)
        {
            setNow();

            setScales(orders, trades);

            drawOrders(orders);
            drawTrades(trades);
        }

        function startIntervals()
        {
            // redraw circles over time with new y axis
            setInterval(function()
            {
                redraw(orders, trades);
            }, redrawInterval);


            //repeatedly load order book
            setInterval(function()
            {
                getData(function(error, orders, trades)
                {
                    redraw(orders, trades);
                });
            }, 3000);
        }
    </script>
  </head>
<body>
  <script type="text/javascript">

  renderChart();

  getData(function(error, orders, trades)
  {
      redraw(orders, trades);

      renderTimeAxis(time_scale);
      renderPriceAxis(price_scale);
  });

  </script>

  <button id="start" onclick="
    realStartTime = Date.now();
    startIntervals();">start</button>
</body>
</html>
