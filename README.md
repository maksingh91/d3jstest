# d3jstest


'use strict';
angular.module('fittingApp')
    .directive('sPersion', function() {

        return {
            restrict: 'A',
            template: '<div id="graphDisPersion"></div>',
            //  template: '<div ng-if="dataAvail" id="graphDisPersion"></div><svg  ng-if="!dataAvail"  height="300px" width="100%" class="nvd3-svg" style="height: 300px; width: 100%;"><text class="nvd3 nv-noData" dy="-.7em" x="586.5" y="140" style="text-anchor: middle;">No Data Available.</text></svg>',

            link: function(scope, element, iAttrs) {

                console.log('iAttrs.sPersion', iAttrs.sPersion);

                scope.$watch(iAttrs.sPersion, function(newValue) {
                    d3.select('svg').html("");
                    console.log("newValue", newValue);
                    if (newValue) {

                        //d3.select("#graphDisPersion").html("");


                        /* created the container */
                        var svgContainer = d3.select("#graphDisPersion").append("svg")
                            .attr("width", '100%')
                            .attr("height", '300px');

                        if (newValue.length == 0) {
                            svgContainer.append("text")
                                .attr("x", "50%")
                                .attr("y", "50%")
                                .attr("alignment-baseline", "middle")
                                .attr("class", "nvd3 nv-noData")
                                .attr("text-anchor", "middle")
                                .text('No Data Available.');
                        }
                        var width = document.getElementById("graphDisPersion").clientWidth;
                        var height = document.getElementById("graphDisPersion").clientHeight;

                        /* create graph points*/
                        //  scope.dataAvail=false;
                        var dataAttr = [];
                        var maxDataPointX = 0;
                        var graphDevisons = 12;
                        var graphRange = 0;
                        //  var initialScaleDataX = [];
                        //  var initialScaleDataY = [];
                        var rangeAllowedX = 1020;
                        var dividerX = 0;
                        var linearScaleX;
                        var linearScaleY;
                        var dataX = [];
                        var dataY = [];
                        var minLinerVal = 0;
                        var maxLinerVal = 0;
                        var enclosingRadious = 0;
                        var containerCircle = 0;
                        var dividerY = 0;
                        var numberAxisX = 20;
                        var xScaleDivider = 0;
                        var yScaleDivider = 0;
                        var dataArrayList = [];

                        for (var i in newValue) {
                            var shotArray = [];
                            var initialScaleDataX = [];
                            var initialScaleDataY = [];
                            //var linearScaleDataX = [];
                            //var linearScaleDataY = [];

                            if (newValue[i].length > 0) {
                                for (var j in newValue[i]) {

                                    if (newValue[i][j].hasOwnProperty('shotGraphs') && newValue[i][j].shotGraphs.length > 0) {

                                        shotArray.push({
                                            x: newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].x,
                                            y: newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].z,
                                            r: "5",
                                            fill: newValue[i][j].color ? newValue[i][j].color.hexCode : "#063252"
                                        });
                                        initialScaleDataX.push(newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].x);
                                        initialScaleDataY.push(newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].z);
                                        dataX.push(newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].x);
                                        dataY.push(newValue[i][j].shotGraphs[newValue[i][j].shotGraphs.length - 1].z);

                                    }
                                }

                                getChartPlotData(shotArray, newValue[i][j].color ? newValue[i][j].color.hexCode : "#063252");

                                dataArrayList.push({
                                    shotGraphPoints: shotArray,
                                    color: newValue[i][j].color ? newValue[i][j].color.hexCode : "#063252"
                                });
                            }
                        }

                        if (dataArrayList.length) {

                            for (var k in dataArrayList) {

                                var arraylist = dataArrayList[k].shotGraphPoints;

                                if (arraylist.length > 0) {
                                    for (var l in arraylist) {
                                        if (linearScaleX) {
                                            arraylist[l].x = linearScaleX(arraylist[l].x);
                                        }
                                        if (linearScaleY) {
                                            arraylist[l].y = linearScaleY(arraylist[l].y);
                                        }

                                    }

                                    if (arraylist.length > 1) {
                                        getOuterCircle(arraylist, dataArrayList[k].color);
                                        arraylist.push(containerCircle);
                                    }

                                    dataAttr = dataAttr.concat(arraylist)

                                }
                            }


                            /* arc draw */
                            var arc = d3.svg.arc()
                                .innerRadius(function(d) {
                                    return d.r
                                })
                                .outerRadius(function(d) {
                                    return Math.abs(d.r) + 1
                                })
                                .startAngle(0)
                                .endAngle(Math.PI);

                            var array = [];

                            for (var i = 0; i < 9; i++) {
                                var obj = {};
                                obj.y = (width / numberAxisX);
                                obj.x = 0;
                                obj.r = ((i) * ((width) / numberAxisX));
                                obj.stroke = "black";
                                obj.fill = "none";
                                array.push(obj);
                                var gra = svgContainer.append("g").attr("transform", "translate(0,150)")
                                gra.append("path")
                                    .attr("class", "arc").attr('fill', '#fff')
                                    .attr("d", arc(obj)).attr('x', 0);

                                //  if (maxDataPointX * 3 > width) {
                                gra.append("text").text((obj.r * xScaleDivider).toFixed(0)).attr('x', obj.r + 5).attr('y', -4);
                                //  } else {
                                //      gra.append("text").text(obj.r).attr('x', obj.r + 5).attr('y', -4);
                                //  }



                                if (i == 13) {
                                    gra.append("text").text("yds").attr('x', obj.r + 10).attr('y', 14);
                                }

                            };


                            /* circle and point display */
                            var circles = svgContainer.append('g').selectAll("circle")
                                .data(dataAttr)
                                .enter()
                                .append("circle");


                            var circleAttributes = circles
                                .attr("cx", function(d) {
                                    return Math.abs(d.x);
                                })
                                .attr("cy", function(d) {
                                    return d.y + 150;
                                })
                                .attr("r", function(d) {
                                    return d.r;
                                })
                                .attr("style", function(d) {
                                    var stroke = "";
                                    if (d.fill == 'none') {
                                        stroke = "stroke: " + d.stroke + ";stroke-width: 1;";
                                    }
                                    return stroke + "fill:" + d.fill;
                                });


                            /* x axis line */
                            var line = svgContainer.append("line").attr('x1', 1).attr('y1', height / 2).attr('x2', width).attr('y2', height / 2).attr("stroke", "#fff").attr("stroke-width", "1px");


                            /* create show yellow points */
                            //  scope.$watch("selectedRow", function() {
                            //        for (var i = 0; i < element.find("circle").length - 1; i++) {
                            //          element.find("circle").eq(i).attr("style", "fill:newValue[1].color");
                            //      }

                            //      element.find("circle").eq(scope.selectedRow).attr("style", "fill:newValue[1].color");
                            //  });
                        }

                        function getChartPlotData(shotArray, color) {

                            var minDataPointX = d3.min(initialScaleDataX);
                            var maxDataPointX = d3.max(initialScaleDataX);
                            var minDataPointY = d3.min(initialScaleDataY);
                            var maxDataPointY = d3.max(initialScaleDataY);
                            if (shotArray.length > 1) {
                                getOuterCircle(shotArray, color);
                            }
                            //dataAttr.push(containerCircle);

                            var maxDispersion = d3.max([Math.abs(maxDataPointY), Math.abs(containerCircle.y) + containerCircle.r]);
                            var maxDistance = d3.max([maxDataPointX, Math.abs(containerCircle.x) + containerCircle.r]);

                            var scaleX = 1;
                            var fromDistance = (width / (maxDistance + 50));
                            if (fromDistance * 2 * (maxDispersion + 10) < width) {
                                scaleX = width / (maxDistance + 50);
                            } else {
                                scaleX = height / ((maxDispersion + 10) * 2);
                            }

                            numberAxisX = 8;

                            var delta = Math.round(((width) / numberAxisX) / scaleX);
                            var calculateDistanceAxis = parseInt(Math.round(delta / 10));
                            var constantsAxis = d3.max([20, calculateDistanceAxis * 10]);

                            dividerX = constantsAxis / (width / (numberAxisX)); // 90*? = 80

                            // var diffX = (maxDataPointX / dividerX - minDataPointX / dividerX);
                            //if(diffX<20)
                            // {
                            //    dividerX = diffX/20;
                            //}
                            // }

                            var scaleY = 1;
                            var fromDistanceY = (height / (maxDispersion + 10));
                            if (fromDistanceY * 2 * (maxDistance + 50) < height) {
                                scaleY = height / (maxDispersion + 10);
                            } else {
                                scaleY = width / ((maxDistance + 50) * 2);
                            }

                            /*    if(xScaleDivider <= dividerX)
                                {
                                  linearScaleX = d3.scale.linear()
                                    .domain([d3.min(dataX), d3.max(dataX)])
                                    .range([d3.min(dataX) / dividerX, d3.max(dataX) / dividerX]);
                                  xScaleDivider = dividerX;
                                } */

                            var numberAxisY = 8;

                            var deltaY = Math.round(((width) / numberAxisY) / scaleY);
                            var calculateDistanceAxisY = parseInt(Math.round(deltaY / 10));
                            var constantsAxisY = d3.max([5, calculateDistanceAxisY * 10]);
                            dividerY = constantsAxisY / (height / numberAxisY);
                            var diff = (maxDataPointY / dividerY - minDataPointY / dividerY);
                            if (diff < 20 && diff > 0) {
                                dividerY = diff;
                            }

                            if (xScaleDivider <= dividerX) {
                                linearScaleX = d3.scale.linear()
                                    .domain([d3.min(dataX), d3.max(dataX)])
                                    .range([d3.min(dataX) / dividerX, d3.max(dataX) / dividerX]);
                                xScaleDivider = dividerX;
                            }

                            // var diff = (maxDataPointY / dividerY - minDataPointY / dividerY);
                            // if (diff<20 && diff > 0) {
                            //     dividerY = diff / 50;
                            // }


                            //  var diff = (maxDataPointY / dividerY - minDataPointY / dividerY);
                            //  if(diff<20 && diff > 0)
                            //  {
                            //    dividerY = diff/20;
                            //  }
                            if (yScaleDivider <= dividerY) {

                                linearScaleY = d3.scale.linear()
                                    .domain([d3.min(dataY), d3.max(dataY)])
                                    .range([d3.min(dataY) / dividerY, d3.max(dataY) / dividerY]);
                                yScaleDivider = dividerY;

                                console.log("changes");
                            }
                        }

                        function getOuterCircle(dataList, color) {
                            //     if(parseInt(dataList[0].y)  < Math.round(100)){
                            //         dataList[0].y = parseInt(dataList[0].y) + 150;
                            // }
                            containerCircle = makeCircle(dataList);
                            containerCircle.r = containerCircle.r + 5;
                            containerCircle.fill = 'none';
                            containerCircle.stroke = color;
                        }
                    }
                }, true);
            }



        }
    });




#json

[
  {
    "id": 82,
    "number": 1,
    "ballMph": 180,
    "launchAngle": 18,
    "verticalRpm": 2222,
    "horizontalRpm": 800,
    "carryYards": null,
    "totalYards": null,
    "landingAngle": 0,
    "deviationAngle": 40,
    "smashFactor": null,
    "ignore": false,
    "manuallyEntered": true,
    "rdPeakHeightYards": 44.26819430850242,
    "rdCarryYards": 282.1717188213406,
    "rdRollYards": 296.2514546757613,
    "rdDevCarryYards": 218.3131906569449,
    "rdDevRollYards": 229.8058691769819,
    "rdLandingDeg": -44.35193186480763,
    "carryIdx": 68,
    "rollIdx": 265,
    "clubConfigId": 54,
    "average": false,
    "shotGraphs": [
      {
        "id": 15809,
        "t": 0,
        "x": 0,
        "y": 0,
        "z": 0,
        "shotId": 82
      },
      {
        "id": 15810,
        "t": 0.1,
        "x": 9.931309731827975,
        "y": 2.681710809870178,
        "z": 5.315785196412944,
        "shotId": 82
      },
      {
        "id": 15811,
        "t": 0.2,
        "x": 19.42710324058674,
        "y": 5.28656462042389,
        "z": 10.5081515916054,
        "shotId": 82
      },
      {
        "id": 15812,
        "t": 0.3,
        "x": 28.51531603401251,
        "y": 7.812555027080468,
        "z": 15.58327484120715,
        "shotId": 82
      },
      {
        "id": 15813,
        "t": 0.4,
        "x": 37.22156676181979,
        "y": 10.25803097806001,
        "z": 20.54687267406322,
        "shotId": 82
      },
      {
        "id": 15814,
        "t": 0.5,
        "x": 45.56943026139181,
        "y": 12.62164486747954,
        "z": 25.40425961782156,
        "shotId": 82
      },
      {
        "id": 15815,
        "t": 0.6,
        "x": 53.58068428022507,
        "y": 14.90223100773633,
        "z": 30.160335239205,
        "shotId": 82
      },
      {
        "id": 15816,
        "t": 0.7,
        "x": 61.2759443244544,
        "y": 17.09864319621142,
        "z": 34.81967627319703,
        "shotId": 82
      },
      {
        "id": 15817,
        "t": 0.7999999999999999,
        "x": 68.67482653640053,
        "y": 19.20982350152218,
        "z": 39.38664847119334,
        "shotId": 82
      },
      {
        "id": 15818,
        "t": 0.8999999999999999,
        "x": 75.79568339995802,
        "y": 21.23490889243963,
        "z": 43.86539864849652,
        "shotId": 82
      },
      {
        "id": 15819,
        "t": 0.9999999999999999,
        "x": 82.65534039565898,
        "y": 23.17330484495466,
        "z": 48.25981637119938,
        "shotId": 82
      },
      {
        "id": 15820,
        "t": 1.1,
        "x": 89.26928265495195,
        "y": 25.02462725510644,
        "z": 52.57355415638639,
        "shotId": 82
      },
      {
        "id": 15821,
        "t": 1.2,
        "x": 95.65193995130429,
        "y": 26.7886328871538,
        "z": 56.81007501260591,
        "shotId": 82
      },
      {
        "id": 15822,
        "t": 1.3,
        "x": 101.8167430207773,
        "y": 28.46520858045965,
        "z": 60.97266176622645,
        "shotId": 82
      },
      {
        "id": 15823,
        "t": 1.4,
        "x": 107.7761688190467,
        "y": 30.05435602281056,
        "z": 65.06441665707405,
        "shotId": 82
      },
      {
        "id": 15824,
        "t": 1.5,
        "x": 113.5416648126017,
        "y": 31.5561878565899,
        "z": 69.08821628976845,
        "shotId": 82
      },
      {
        "id": 15825,
        "t": 1.6,
        "x": 119.1235065557641,
        "y": 32.97096554547098,
        "z": 73.04666868983549,
        "shotId": 82
      },
      {
        "id": 15826,
        "t": 1.7,
        "x": 124.5307888485969,
        "y": 34.29911360395933,
        "z": 76.94210380031456,
        "shotId": 82
      },
      {
        "id": 15827,
        "t": 1.8,
        "x": 129.7715421352307,
        "y": 35.54119886292368,
        "z": 80.7765846348109,
        "shotId": 82
      },
      {
        "id": 15828,
        "t": 1.900000000000001,
        "x": 134.8530203010647,
        "y": 36.69786907646265,
        "z": 84.5519638886192,
        "shotId": 82
      },
      {
        "id": 15829,
        "t": 2,
        "x": 139.7819015537682,
        "y": 37.76981193012253,
        "z": 88.26992796931165,
        "shotId": 82
      },
      {
        "id": 15830,
        "t": 2.100000000000001,
        "x": 144.5644337583664,
        "y": 38.75772516230306,
        "z": 91.93202666970893,
        "shotId": 82
      },
      {
        "id": 15831,
        "t": 2.200000000000001,
        "x": 149.2065679149569,
        "y": 39.66228890753851,
        "z": 95.53970550952262,
        "shotId": 82
      },
      {
        "id": 15832,
        "t": 2.300000000000001,
        "x": 153.7139720476525,
        "y": 40.48416259427172,
        "z": 99.0943022605367,
        "shotId": 82
      },
      {
        "id": 15833,
        "t": 2.400000000000001,
        "x": 158.0921544942705,
        "y": 41.22395836995566,
        "z": 102.5970823140413,
        "shotId": 82
      },
      {
        "id": 15834,
        "t": 2.500000000000001,
        "x": 162.3465042821794,
        "y": 41.88224132914664,
        "z": 106.0492606089255,
        "shotId": 82
      },
      {
        "id": 15835,
        "t": 2.600000000000001,
        "x": 166.4823118964462,
        "y": 42.45952478704719,
        "z": 109.4520077785462,
        "shotId": 82
      },
      {
        "id": 15836,
        "t": 2.700000000000001,
        "x": 170.5047623209354,
        "y": 42.95628359580679,
        "z": 112.8064603966848,
        "shotId": 82
      },
      {
        "id": 15837,
        "t": 2.800000000000001,
        "x": 174.41894347342,
        "y": 43.37294845977231,
        "z": 116.1137210149415,
        "shotId": 82
      },
      {
        "id": 15838,
        "t": 2.900000000000001,
        "x": 178.2298228505227,
        "y": 43.70992420258628,
        "z": 119.3748626777487,
        "shotId": 82
      },
      {
        "id": 15839,
        "t": 3.000000000000001,
        "x": 181.9422530872076,
        "y": 43.9675870463323,
        "z": 122.5909314035284,
        "shotId": 82
      },
      {
        "id": 15840,
        "t": 3.100000000000001,
        "x": 185.5609616625451,
        "y": 44.14628830601548,
        "z": 125.7629429023887,
        "shotId": 82
      },
      {
        "id": 15841,
        "t": 3.200000000000002,
        "x": 189.090518146899,
        "y": 44.24638103815965,
        "z": 128.8918868512598,
        "shotId": 82
      },
      {
        "id": 15842,
        "t": 3.300000000000002,
        "x": 192.5353612794343,
        "y": 44.26819430850242,
        "z": 131.9787202123395,
        "shotId": 82
      },
      {
        "id": 15843,
        "t": 3.400000000000002,
        "x": 195.8997662154038,
        "y": 44.21206167872324,
        "z": 135.0243695078383,
        "shotId": 82
      },
      {
        "id": 15844,
        "t": 3.500000000000002,
        "x": 199.1878261840679,
        "y": 44.0783347173491,
        "z": 138.0297260528048,
        "shotId": 82
      },
      {
        "id": 15845,
        "t": 3.600000000000002,
        "x": 202.4034517525689,
        "y": 43.86736445147702,
        "z": 140.995620072292,
        "shotId": 82
      },
      {
        "id": 15846,
        "t": 3.700000000000002,
        "x": 205.550379778147,
        "y": 43.57951871810619,
        "z": 143.9228418640894,
        "shotId": 82
      },
      {
        "id": 15847,
        "t": 3.800000000000002,
        "x": 208.6321869854201,
        "y": 43.21518986490054,
        "z": 146.812157653635,
        "shotId": 82
      },
      {
        "id": 15848,
        "t": 3.900000000000002,
        "x": 211.6523054883475,
        "y": 42.77477959307397,
        "z": 149.6643055379598,
        "shotId": 82
      },
      {
        "id": 15849,
        "t": 4.000000000000002,
        "x": 214.6139992374228,
        "y": 42.25871014035555,
        "z": 152.4799754106475,
        "shotId": 82
      },
      {
        "id": 15850,
        "t": 4.100000000000001,
        "x": 217.5203791986838,
        "y": 41.66743501390036,
        "z": 155.2598271403122,
        "shotId": 82
      },
      {
        "id": 15851,
        "t": 4.200000000000001,
        "x": 220.3744163944225,
        "y": 41.00143886209361,
        "z": 158.00449639348,
        "shotId": 82
      },
      {
        "id": 15852,
        "t": 4.300000000000001,
        "x": 223.1789501073203,
        "y": 40.26123619041053,
        "z": 160.7145945499656,
        "shotId": 82
      },
      {
        "id": 15853,
        "t": 4.4,
        "x": 225.9366958686721,
        "y": 39.44737027014446,
        "z": 163.3907088611568,
        "shotId": 82
      },
      {
        "id": 15854,
        "t": 4.5,
        "x": 228.6502494584863,
        "y": 38.56041412337624,
        "z": 166.0333995032466,
        "shotId": 82
      },
      {
        "id": 15855,
        "t": 4.6,
        "x": 231.3220916928738,
        "y": 37.60097142775626,
        "z": 168.6431975982958,
        "shotId": 82
      },
      {
        "id": 15856,
        "t": 4.699999999999999,
        "x": 233.9546032006736,
        "y": 36.56967211411215,
        "z": 171.2206125938287,
        "shotId": 82
      },
      {
        "id": 15857,
        "t": 4.799999999999999,
        "x": 236.5500723439362,
        "y": 35.46717090858282,
        "z": 173.7661342067074,
        "shotId": 82
      },
      {
        "id": 15858,
        "t": 4.899999999999999,
        "x": 239.110701360965,
        "y": 34.29414661939771,
        "z": 176.2802332177677,
        "shotId": 82
      },
      {
        "id": 15859,
        "t": 4.999999999999998,
        "x": 241.6386121792233,
        "y": 33.05130143365373,
        "z": 178.7633623734676,
        "shotId": 82
      },
      {
        "id": 15860,
        "t": 5.099999999999998,
        "x": 244.135851884795,
        "y": 31.73936020617673,
        "z": 181.2159573728361,
        "shotId": 82
      },
      {
        "id": 15861,
        "t": 5.199999999999998,
        "x": 246.6043978391079,
        "y": 30.35906972628392,
        "z": 183.6384379193266,
        "shotId": 82
      },
      {
        "id": 15862,
        "t": 5.299999999999997,
        "x": 249.0461624375653,
        "y": 28.91119795175426,
        "z": 186.0312088185619,
        "shotId": 82
      },
      {
        "id": 15863,
        "t": 5.399999999999997,
        "x": 251.4629975085369,
        "y": 27.39653320252336,
        "z": 188.3946611044093,
        "shotId": 82
      },
      {
        "id": 15864,
        "t": 5.499999999999996,
        "x": 253.8566983547912,
        "y": 25.81588330951648,
        "z": 190.7291731773335,
        "shotId": 82
      },
      {
        "id": 15865,
        "t": 5.599999999999996,
        "x": 256.2290074428425,
        "y": 24.17007471659349,
        "z": 193.035111940535,
        "shotId": 82
      },
      {
        "id": 15866,
        "t": 5.699999999999996,
        "x": 258.5816177488016,
        "y": 22.45995153579241,
        "z": 195.3128339209717,
        "shotId": 82
      },
      {
        "id": 15867,
        "t": 5.799999999999995,
        "x": 260.9161757721047,
        "y": 20.68637455791967,
        "z": 197.5626863639598,
        "shotId": 82
      },
      {
        "id": 15868,
        "t": 5.899999999999995,
        "x": 263.2342842309221,
        "y": 18.85022022205299,
        "z": 199.7850082916328,
        "shotId": 82
      },
      {
        "id": 15869,
        "t": 5.999999999999995,
        "x": 265.5375044550933,
        "y": 16.95237954871238,
        "z": 201.9801315170889,
        "shotId": 82
      },
      {
        "id": 15870,
        "t": 6.099999999999994,
        "x": 267.8273584940905,
        "y": 14.99375704233742,
        "z": 204.1483816075431,
        "shotId": 82
      },
      {
        "id": 15871,
        "t": 6.199999999999994,
        "x": 270.1053309587713,
        "y": 12.97526956931297,
        "z": 206.2900787912182,
        "shotId": 82
      },
      {
        "id": 15872,
        "t": 6.299999999999994,
        "x": 272.3728700446177,
        "y": 10.89784619352948,
        "z": 208.4055380782447,
        "shotId": 82
      },
      {
        "id": 15873,
        "t": 6.399999999999993,
        "x": 274.6313860730075,
        "y": 8.762432047082159,
        "z": 210.495066430399,
        "shotId": 82
      },
      {
        "id": 15874,
        "t": 6.499999999999993,
        "x": 276.8822543004052,
        "y": 6.569984686982542,
        "z": 212.5589655741306,
        "shotId": 82
      },
      {
        "id": 15875,
        "t": 6.599999999999993,
        "x": 279.1268175300769,
        "y": 4.321470182679686,
        "z": 214.5975347767132,
        "shotId": 82
      },
      {
        "id": 15876,
        "t": 6.699999999999992,
        "x": 281.3663869144797,
        "y": 2.01786193364881,
        "z": 216.6110714781886,
        "shotId": 82
      },
      {
        "id": 15877,
        "t": 6.785585220005941,
        "x": 283.2799489260338,
        "y": 0,
        "z": 218.3131906569449,
        "shotId": 82
      },
      {
        "id": 15878,
        "t": 6.785585220005941,
        "x": 283.2799489260338,
        "y": 0,
        "z": 218.3131906569449,
        "shotId": 82
      },
      {
        "id": 15879,
        "t": 6.80558522000594,
        "x": 283.3779709096916,
        "y": 0.1180311585322778,
        "z": 218.4000375633728,
        "shotId": 82
      },
      {
        "id": 15880,
        "t": 6.825585220005941,
        "x": 283.4759832342338,
        "y": 0.2316729929535328,
        "z": 218.4868759118805,
        "shotId": 82
      },
      {
        "id": 15881,
        "t": 6.84558522000594,
        "x": 283.5739845239805,
        "y": 0.3409299197991928,
        "z": 218.5737044836237,
        "shotId": 82
      },
      {
        "id": 15882,
        "t": 6.865585220005941,
        "x": 283.6719734658313,
        "y": 0.4458062197757791,
        "z": 218.6605221152032,
        "shotId": 82
      },
      {
        "id": 15883,
        "t": 6.88558522000594,
        "x": 283.7699488068536,
        "y": 0.5463060384931175,
        "z": 218.7473276965277,
        "shotId": 82
      },
      {
        "id": 15884,
        "t": 6.905585220005941,
        "x": 283.8679093517194,
        "y": 0.642433387259389,
        "z": 218.8341201685439,
        "shotId": 82
      },
      {
        "id": 15885,
        "t": 6.92558522000594,
        "x": 283.9658539599792,
        "y": 0.7341921439515796,
        "z": 218.9208985208199,
        "shotId": 82
      },
      {
        "id": 15886,
        "t": 6.945585220005941,
        "x": 284.0637815431581,
        "y": 0.8215860539755719,
        "z": 219.0076617889735,
        "shotId": 82
      },
      {
        "id": 15887,
        "t": 6.96558522000594,
        "x": 284.161691061666,
        "y": 0.9046187313319405,
        "z": 219.0944090519339,
        "shotId": 82
      },
      {
        "id": 15888,
        "t": 6.985585220005941,
        "x": 284.2595815215059,
        "y": 0.9832936598054381,
        "z": 219.1811394290254,
        "shotId": 82
      },
      {
        "id": 15889,
        "t": 7.005585220005941,
        "x": 284.3574519707706,
        "y": 1.057614194298171,
        "z": 219.2678520768638,
        "shotId": 82
      },
      {
        "id": 15890,
        "t": 7.025585220005941,
        "x": 284.455301495918,
        "y": 1.127583562328492,
        "z": 219.3545461860563,
        "shotId": 82
      },
      {
        "id": 15891,
        "t": 7.045585220005941,
        "x": 284.5531292178153,
        "y": 1.193204865719622,
        "z": 219.4412209776961,
        "shotId": 82
      },
      {
        "id": 15892,
        "t": 7.065585220005941,
        "x": 284.6509342875457,
        "y": 1.254481082503877,
        "z": 219.5278756996481,
        "shotId": 82
      },
      {
        "id": 15893,
        "t": 7.085585220005941,
        "x": 284.7487158819741,
        "y": 1.311415069069957,
        "z": 219.6145096226193,
        "shotId": 82
      },
      {
        "id": 15894,
        "t": 7.105585220005941,
        "x": 284.8464731990708,
        "y": 1.364009562581955,
        "z": 219.701122036016,
        "shotId": 82
      },
      {
        "id": 15895,
        "t": 7.125585220005941,
        "x": 284.9442054529966,
        "y": 1.412267183699398,
        "z": 219.7877122435894,
        "shotId": 82
      },
      {
        "id": 15896,
        "t": 7.145585220005941,
        "x": 285.0419118689593,
        "y": 1.456190439627495,
        "z": 219.874279558878,
        "shotId": 82
      },
      {
        "id": 15897,
        "t": 7.165585220005941,
        "x": 285.1395916778531,
        "y": 1.495781727525757,
        "z": 219.9608233004585,
        "shotId": 82
      },
      {
        "id": 15898,
        "t": 7.185585220005941,
        "x": 285.237244110703,
        "y": 1.531043338300982,
        "z": 220.0473427870238,
        "shotId": 82
      },
      {
        "id": 15899,
        "t": 7.205585220005941,
        "x": 285.3348683929401,
        "y": 1.561977460807149,
        "z": 220.1338373323099,
        "shotId": 82
      },
      {
        "id": 15900,
        "t": 7.225585220005941,
        "x": 285.4324637385386,
        "y": 1.588586186469984,
        "z": 220.2203062399029,
        "shotId": 82
      },
      {
        "id": 15901,
        "t": 7.245585220005941,
        "x": 285.5300293440579,
        "y": 1.610871514347676,
        "z": 220.306748797959,
        "shotId": 82
      },
      {
        "id": 15902,
        "t": 7.265585220005941,
        "x": 285.6275643826306,
        "y": 1.628835356631618,
        "z": 220.3931642738785,
        "shotId": 82
      },
      {
        "id": 15903,
        "t": 7.285585220005941,
        "x": 285.7250679979481,
        "y": 1.642479544582218,
        "z": 220.4795519089771,
        "shotId": 82
      },
      {
        "id": 15904,
        "t": 7.305585220005941,
        "x": 285.8225392982984,
        "y": 1.651805834885077,
        "z": 220.5659109132029,
        "shotId": 82
      },
      {
        "id": 15905,
        "t": 7.325585220005941,
        "x": 285.9199773507088,
        "y": 1.656815916402574,
        "z": 220.6522404599477,
        "shotId": 82
      },
      {
        "id": 15906,
        "t": 7.345585220005941,
        "x": 286.0173811752523,
        "y": 1.657511417285605,
        "z": 220.7385396810018,
        "shotId": 82
      },
      {
        "id": 15907,
        "t": 7.365585220005941,
        "x": 286.1147497395677,
        "y": 1.653893912400471,
        "z": 220.8248076616993,
        "shotId": 82
      },
      {
        "id": 15908,
        "t": 7.385585220005941,
        "x": 286.212081953647,
        "y": 1.645964931017273,
        "z": 220.9110434362994,
        "shotId": 82
      },
      {
        "id": 15909,
        "t": 7.405585220005941,
        "x": 286.3093766649317,
        "y": 1.633725964699101,
        "z": 220.9972459836418,
        "shotId": 82
      },
      {
        "id": 15910,
        "t": 7.425585220005941,
        "x": 286.4066326537537,
        "y": 1.617178475326322,
        "z": 221.083414223108,
        "shotId": 82
      },
      {
        "id": 15911,
        "t": 7.445585220005941,
        "x": 286.5038486291538,
        "y": 1.596323903187543,
        "z": 221.1695470109155,
        "shotId": 82
      },
      {
        "id": 15912,
        "t": 7.465585220005941,
        "x": 286.6010232250913,
        "y": 1.571163675068536,
        "z": 221.2556431367603,
        "shotId": 82
      },
      {
        "id": 15913,
        "t": 7.485585220005941,
        "x": 286.6981549970591,
        "y": 1.541699212272454,
        "z": 221.3417013208174,
        "shotId": 82
      },
      {
        "id": 15914,
        "t": 7.505585220005941,
        "x": 286.7952424191041,
        "y": 1.507931938508773,
        "z": 221.4277202111012,
        "shotId": 82
      },
      {
        "id": 15915,
        "t": 7.525585220005941,
        "x": 286.8922838812458,
        "y": 1.469863287594326,
        "z": 221.5136983811786,
        "shotId": 82
      },
      {
        "id": 15916,
        "t": 7.545585220005941,
        "x": 286.9892776872812,
        "y": 1.427494710916945,
        "z": 221.5996343282234,
        "shotId": 82
      },
      {
        "id": 15917,
        "t": 7.565585220005941,
        "x": 287.0862220529544,
        "y": 1.380827684620267,
        "z": 221.6855264713953,
        "shotId": 82
      },
      {
        "id": 15918,
        "t": 7.585585220005941,
        "x": 287.183115104469,
        "y": 1.329863716476625,
        "z": 221.7713731505216,
        "shotId": 82
      },
      {
        "id": 15919,
        "t": 7.605585220005941,
        "x": 287.2799548773161,
        "y": 1.274604352423306,
        "z": 221.857172625059,
        "shotId": 82
      },
      {
        "id": 15920,
        "t": 7.625585220005942,
        "x": 287.3767393153896,
        "y": 1.215051182745378,
        "z": 221.9429230733093,
        "shotId": 82
      },
      {
        "id": 15921,
        "t": 7.645585220005941,
        "x": 287.4734662703594,
        "y": 1.151205847895554,
        "z": 222.0286225918647,
        "shotId": 82
      },
      {
        "id": 15922,
        "t": 7.665585220005942,
        "x": 287.5701335012748,
        "y": 1.083070043948043,
        "z": 222.1142691952559,
        "shotId": 82
      },
      {
        "id": 15923,
        "t": 7.685585220005941,
        "x": 287.666738674369,
        "y": 1.010645527688812,
        "z": 222.1998608157789,
        "shotId": 82
      },
      {
        "id": 15924,
        "t": 7.705585220005942,
        "x": 287.7632793630399,
        "y": 0.9339341213492782,
        "z": 222.2853953034787,
        "shotId": 82
      },
      {
        "id": 15925,
        "t": 7.725585220005941,
        "x": 287.8597530479832,
        "y": 0.8529377169940335,
        "z": 222.3708704262663,
        "shotId": 82
      },
      {
        "id": 15926,
        "t": 7.745585220005942,
        "x": 287.9561571174542,
        "y": 0.7676582805760231,
        "z": 222.4562838701509,
        "shotId": 82
      },
      {
        "id": 15927,
        "t": 7.765585220005941,
        "x": 288.0524888676395,
        "y": 0.6780978556745179,
        "z": 222.5416332395697,
        "shotId": 82
      },
      {
        "id": 15928,
        "t": 7.785585220005942,
        "x": 288.1487455031221,
        "y": 0.5842585669325737,
        "z": 222.6269160577996,
        "shotId": 82
      },
      {
        "id": 15929,
        "t": 7.805585220005941,
        "x": 288.2449241374228,
        "y": 0.486142623211384,
        "z": 222.7121297674367,
        "shotId": 82
      },
      {
        "id": 15930,
        "t": 7.825585220005941,
        "x": 288.3410217936049,
        "y": 0.3837523204792077,
        "z": 222.7972717309327,
        "shotId": 82
      },
      {
        "id": 15931,
        "t": 7.845585220005941,
        "x": 288.437035404932,
        "y": 0.2770900444524528,
        "z": 222.8823392311767,
        "shotId": 82
      },
      {
        "id": 15932,
        "t": 7.865585220005942,
        "x": 288.5329618155674,
        "y": 0.1661582730061059,
        "z": 222.9673294721159,
        "shotId": 82
      },
      {
        "id": 15933,
        "t": 7.885585220005941,
        "x": 288.628797781308,
        "y": 0.05095957837009676,
        "z": 223.0522395794053,
        "shotId": 82
      },
      {
        "id": 15934,
        "t": 7.894116664892102,
        "x": 288.6696387417607,
        "y": 0,
        "z": 223.0884244324198,
        "shotId": 82
      },
      {
        "id": 15935,
        "t": 7.894116664892102,
        "x": 288.6696387417607,
        "y": 0,
        "z": 223.0884244324198,
        "shotId": 82
      },
      {
        "id": 15936,
        "t": 7.914116664892101,
        "x": 288.7519025899506,
        "y": 0.04225524607110799,
        "z": 223.1613097226329,
        "shotId": 82
      },
      {
        "id": 15937,
        "t": 7.934116664892102,
        "x": 288.8341483298532,
        "y": 0.08018987750730085,
        "z": 223.2341789690087,
        "shotId": 82
      },
      {
        "id": 15938,
        "t": 7.954116664892101,
        "x": 288.9163753112922,
        "y": 0.1138059249736559,
        "z": 223.3070315954952,
        "shotId": 82
      },
      {
        "id": 15939,
        "t": 7.974116664892102,
        "x": 288.9985828847915,
        "y": 0.1431053133258115,
        "z": 223.37986702666,
        "shotId": 82
      },
      {
        "id": 15940,
        "t": 7.994116664892101,
        "x": 289.080770394837,
        "y": 0.1680898664614153,
        "z": 223.4526846817218,
        "shotId": 82
      },
      {
        "id": 15941,
        "t": 8.014116664892102,
        "x": 289.1629371729436,
        "y": 0.1887613127721588,
        "z": 223.5254839684065,
        "shotId": 82
      },
      {
        "id": 15942,
        "t": 8.034116664892101,
        "x": 289.2450825305903,
        "y": 0.2051212912138939,
        "z": 223.5982642766885,
        "shotId": 82
      },
      {
        "id": 15943,
        "t": 8.054116664892101,
        "x": 289.3272057521036,
        "y": 0.2171713579967963,
        "z": 223.6710249724853,
        "shotId": 82
      },
      {
        "id": 15944,
        "t": 8.074116664892102,
        "x": 289.4093060875737,
        "y": 0.2249129938794012,
        "z": 223.7437653913811,
        "shotId": 82
      },
      {
        "id": 15945,
        "t": 8.094116664892102,
        "x": 289.4913827458953,
        "y": 0.2283476120305134,
        "z": 223.8164848324613,
        "shotId": 82
      },
      {
        "id": 15946,
        "t": 8.114116664892101,
        "x": 289.5734348880253,
        "y": 0.2274765664027523,
        "z": 223.8891825523387,
        "shotId": 82
      },
      {
        "id": 15947,
        "t": 8.134116664892101,
        "x": 289.6554616205467,
        "y": 0.2223011605423184,
        "z": 223.9618577594508,
        "shotId": 82
      },
      {
        "id": 15948,
        "t": 8.154116664892102,
        "x": 289.7374619896181,
        "y": 0.2128226567430077,
        "z": 224.0345096086998,
        "shotId": 82
      },
      {
        "id": 15949,
        "t": 8.174116664892102,
        "x": 289.8194349753779,
        "y": 0.1990422854399324,
        "z": 224.1071371964943,
        "shotId": 82
      },
      {
        "id": 15950,
        "t": 8.194116664892102,
        "x": 289.9013794868573,
        "y": 0.180961254730842,
        "z": 224.1797395562422,
        "shotId": 82
      },
      {
        "id": 15951,
        "t": 8.214116664892101,
        "x": 289.983294357434,
        "y": 0.1585807599109089,
        "z": 224.2523156543231,
        "shotId": 82
      },
      {
        "id": 15952,
        "t": 8.234116664892102,
        "x": 290.065178340849,
        "y": 0.1319019929102783,
        "z": 224.3248643865586,
        "shotId": 82
      },
      {
        "id": 15953,
        "t": 8.254116664892102,
        "x": 290.1470301077819,
        "y": 0.1009261515320141,
        "z": 224.3973845751787,
        "shotId": 82
      },
      {
        "id": 15954,
        "t": 8.274116664892102,
        "x": 290.2288482429735,
        "y": 0.06565444840030353,
        "z": 224.4698749662719,
        "shotId": 82
      },
      {
        "id": 15955,
        "t": 8.294116664892101,
        "x": 290.310631242866,
        "y": 0.0260881195436753,
        "z": 224.5423342276949,
        "shotId": 82
      },
      {
        "id": 15956,
        "t": 8.30601283713335,
        "x": 290.3592546287766,
        "y": 0,
        "z": 224.5854142643235,
        "shotId": 82
      },
      {
        "id": 15957,
        "t": 8.30601283713335,
        "x": 290.3592546287766,
        "y": 0,
        "z": 224.5854142643235,
        "shotId": 82
      },
      {
        "id": 15958,
        "t": 8.32601283713335,
        "x": 290.4328630753272,
        "y": 0.01855654386294363,
        "z": 224.6506309191119,
        "shotId": 82
      },
      {
        "id": 15959,
        "t": 8.34601283713335,
        "x": 290.5064543030616,
        "y": 0.03280689360791626,
        "z": 224.7158323181293,
        "shotId": 82
      },
      {
        "id": 15960,
        "t": 8.36601283713335,
        "x": 290.5800276662708,
        "y": 0.04275243246570391,
        "z": 224.7810178892817,
        "shotId": 82
      },
      {
        "id": 15961,
        "t": 8.38601283713335,
        "x": 290.6535824837275,
        "y": 0.04839446980901591,
        "z": 224.8461870290054,
        "shotId": 82
      },
      {
        "id": 15962,
        "t": 8.40601283713335,
        "x": 290.7271180309742,
        "y": 0.04973425001301932,
        "z": 224.9113390954351,
        "shotId": 82
      },
      {
        "id": 15963,
        "t": 8.42601283713335,
        "x": 290.8006335329081,
        "y": 0.04677296192909238,
        "z": 224.9764734018347,
        "shotId": 82
      },
      {
        "id": 15964,
        "t": 8.44601283713335,
        "x": 290.8741281567849,
        "y": 0.03951174885916429,
        "z": 225.0415892103971,
        "shotId": 82
      },
      {
        "id": 15965,
        "t": 8.46601283713335,
        "x": 290.9476010057414,
        "y": 0.02795171889478661,
        "z": 225.1066857265072,
        "shotId": 82
      },
      {
        "id": 15966,
        "t": 8.48601283713335,
        "x": 291.0210511129249,
        "y": 0.01209395546957677,
        "z": 225.1717620935388,
        "shotId": 82
      },
      {
        "id": 15967,
        "t": 8.498014126148622,
        "x": 291.065111639321,
        "y": 0,
        "z": 225.2107994632215,
        "shotId": 82
      },
      {
        "id": 15968,
        "t": 8.498014126148622,
        "x": 291.065111639321,
        "y": 0,
        "z": 225.2107994632215,
        "shotId": 82
      },
      {
        "id": 15969,
        "t": 8.518014126148621,
        "x": 291.1333778115959,
        "y": 0.009072199564959303,
        "z": 225.2712828941267,
        "shotId": 82
      },
      {
        "id": 15970,
        "t": 8.53801412614862,
        "x": 291.2016279687716,
        "y": 0.01384354598958161,
        "z": 225.3317521357472,
        "shotId": 82
      },
      {
        "id": 15971,
        "t": 8.558014126148622,
        "x": 291.269861432463,
        "y": 0.01431518589168677,
        "z": 225.3922065870379,
        "shotId": 82
      },
      {
        "id": 15972,
        "t": 8.578014126148622,
        "x": 291.3380774724529,
        "y": 0.01048821203720622,
        "z": 225.4526456010306,
        "shotId": 82
      },
      {
        "id": 15973,
        "t": 8.598014126148621,
        "x": 291.4062752992533,
        "y": 0.002363674148886567,
        "z": 225.5130684782435,
        "shotId": 82
      },
      {
        "id": 15974,
        "t": 8.601820032524337,
        "x": 291.4192493977489,
        "y": 0,
        "z": 225.5245634539212,
        "shotId": 82
      },
      {
        "id": 15975,
        "t": 8.601820032524337,
        "x": 291.4192493977489,
        "y": 0,
        "z": 225.5245634539212,
        "shotId": 82
      },
      {
        "id": 15976,
        "t": 8.632432186122214,
        "x": 291.5168772593469,
        "y": 0,
        "z": 225.6110611705003,
        "shotId": 82
      },
      {
        "id": 15977,
        "t": 8.663044339720091,
        "x": 291.613008308514,
        "y": 0,
        "z": 225.6962327199864,
        "shotId": 82
      },
      {
        "id": 15978,
        "t": 8.693656493317969,
        "x": 291.7081533982025,
        "y": 0,
        "z": 225.7805307151187,
        "shotId": 82
      },
      {
        "id": 15979,
        "t": 8.724268646915846,
        "x": 291.8023125284122,
        "y": 0,
        "z": 225.8639551558973,
        "shotId": 82
      },
      {
        "id": 15980,
        "t": 8.754880800513723,
        "x": 291.8954856991434,
        "y": 0,
        "z": 225.9465060423223,
        "shotId": 82
      },
      {
        "id": 15981,
        "t": 8.7854929541116,
        "x": 291.987672910396,
        "y": 0,
        "z": 226.0281833743935,
        "shotId": 82
      },
      {
        "id": 15982,
        "t": 8.816105107709477,
        "x": 292.0788741621699,
        "y": 0,
        "z": 226.108987152111,
        "shotId": 82
      },
      {
        "id": 15983,
        "t": 8.846717261307354,
        "x": 292.1690894544652,
        "y": 0,
        "z": 226.1889173754749,
        "shotId": 82
      },
      {
        "id": 15984,
        "t": 8.877329414905232,
        "x": 292.2583187872818,
        "y": 0,
        "z": 226.267974044485,
        "shotId": 82
      },
      {
        "id": 15985,
        "t": 8.907941568503109,
        "x": 292.3465621606198,
        "y": 0,
        "z": 226.3461571591414,
        "shotId": 82
      },
      {
        "id": 15986,
        "t": 8.938553722100986,
        "x": 292.4338195744792,
        "y": 0,
        "z": 226.4234667194441,
        "shotId": 82
      },
      {
        "id": 15987,
        "t": 8.969165875698863,
        "x": 292.52009102886,
        "y": 0,
        "z": 226.4999027253931,
        "shotId": 82
      },
      {
        "id": 15988,
        "t": 8.99977802929674,
        "x": 292.605376523762,
        "y": 0,
        "z": 226.5754651769884,
        "shotId": 82
      },
      {
        "id": 15989,
        "t": 9.030390182894617,
        "x": 292.6896760591854,
        "y": 0,
        "z": 226.65015407423,
        "shotId": 82
      },
      {
        "id": 15990,
        "t": 9.061002336492493,
        "x": 292.7729896351303,
        "y": 0,
        "z": 226.7239694171179,
        "shotId": 82
      },
      {
        "id": 15991,
        "t": 9.09161449009037,
        "x": 292.8553172515964,
        "y": 0,
        "z": 226.7969112056521,
        "shotId": 82
      },
      {
        "id": 15992,
        "t": 9.122226643688247,
        "x": 292.936658908584,
        "y": 0,
        "z": 226.8689794398326,
        "shotId": 82
      },
      {
        "id": 15993,
        "t": 9.152838797286124,
        "x": 293.0170146060929,
        "y": 0,
        "z": 226.9401741196594,
        "shotId": 82
      },
      {
        "id": 15994,
        "t": 9.183450950884001,
        "x": 293.0963843441232,
        "y": 0,
        "z": 227.0104952451325,
        "shotId": 82
      },
      {
        "id": 15995,
        "t": 9.214063104481879,
        "x": 293.1747681226748,
        "y": 0,
        "z": 227.0799428162519,
        "shotId": 82
      },
      {
        "id": 15996,
        "t": 9.244675258079756,
        "x": 293.2521659417478,
        "y": 0,
        "z": 227.1485168330175,
        "shotId": 82
      },
      {
        "id": 15997,
        "t": 9.275287411677633,
        "x": 293.3285778013422,
        "y": 0,
        "z": 227.2162172954295,
        "shotId": 82
      },
      {
        "id": 15998,
        "t": 9.30589956527551,
        "x": 293.404003701458,
        "y": 0,
        "z": 227.2830442034878,
        "shotId": 82
      },
      {
        "id": 15999,
        "t": 9.336511718873387,
        "x": 293.478443642095,
        "y": 0,
        "z": 227.3489975571923,
        "shotId": 82
      },
      {
        "id": 16000,
        "t": 9.367123872471264,
        "x": 293.5518976232535,
        "y": 0,
        "z": 227.4140773565433,
        "shotId": 82
      },
      {
        "id": 16001,
        "t": 9.397736026069142,
        "x": 293.6243656449332,
        "y": 0,
        "z": 227.4782836015404,
        "shotId": 82
      },
      {
        "id": 16002,
        "t": 9.428348179667019,
        "x": 293.6958477071345,
        "y": 0,
        "z": 227.5416162921839,
        "shotId": 82
      },
      {
        "id": 16003,
        "t": 9.458960333264896,
        "x": 293.7663438098569,
        "y": 0,
        "z": 227.6040754284736,
        "shotId": 82
      },
      {
        "id": 16004,
        "t": 9.489572486862773,
        "x": 293.8358539531009,
        "y": 0,
        "z": 227.6656610104097,
        "shotId": 82
      },
      {
        "id": 16005,
        "t": 9.52018464046065,
        "x": 293.9043781368662,
        "y": 0,
        "z": 227.7263730379921,
        "shotId": 82
      },
      {
        "id": 16006,
        "t": 9.550796794058527,
        "x": 293.9719163611528,
        "y": 0,
        "z": 227.7862115112207,
        "shotId": 82
      },
      {
        "id": 16007,
        "t": 9.581408947656405,
        "x": 294.0384686259608,
        "y": 0,
        "z": 227.8451764300956,
        "shotId": 82
      },
      {
        "id": 16008,
        "t": 9.612021101254282,
        "x": 294.1040349312901,
        "y": 0,
        "z": 227.9032677946169,
        "shotId": 82
      },
      {
        "id": 16009,
        "t": 9.642633254852159,
        "x": 294.1686152771409,
        "y": 0,
        "z": 227.9604856047844,
        "shotId": 82
      },
      {
        "id": 16010,
        "t": 9.673245408450036,
        "x": 294.2322096635129,
        "y": 0,
        "z": 228.0168298605983,
        "shotId": 82
      },
      {
        "id": 16011,
        "t": 9.703857562047913,
        "x": 294.2948180904064,
        "y": 0,
        "z": 228.0723005620584,
        "shotId": 82
      },
      {
        "id": 16012,
        "t": 9.73446971564579,
        "x": 294.3564405578212,
        "y": 0,
        "z": 228.1268977091649,
        "shotId": 82
      },
      {
        "id": 16013,
        "t": 9.765081869243668,
        "x": 294.4170770657573,
        "y": 0,
        "z": 228.1806213019176,
        "shotId": 82
      },
      {
        "id": 16014,
        "t": 9.795694022841545,
        "x": 294.4767276142149,
        "y": 0,
        "z": 228.2334713403166,
        "shotId": 82
      },
      {
        "id": 16015,
        "t": 9.826306176439422,
        "x": 294.5353922031938,
        "y": 0,
        "z": 228.2854478243619,
        "shotId": 82
      },
      {
        "id": 16016,
        "t": 9.8569183300373,
        "x": 294.593070832694,
        "y": 0,
        "z": 228.3365507540535,
        "shotId": 82
      },
      {
        "id": 16017,
        "t": 9.887530483635175,
        "x": 294.6497635027157,
        "y": 0,
        "z": 228.3867801293915,
        "shotId": 82
      },
      {
        "id": 16018,
        "t": 9.918142637233053,
        "x": 294.7054702132586,
        "y": 0,
        "z": 228.4361359503757,
        "shotId": 82
      },
      {
        "id": 16019,
        "t": 9.948754790830929,
        "x": 294.760190964323,
        "y": 0,
        "z": 228.4846182170062,
        "shotId": 82
      },
      {
        "id": 16020,
        "t": 9.979366944428806,
        "x": 294.8139257559087,
        "y": 0,
        "z": 228.532226929283,
        "shotId": 82
      },
      {
        "id": 16021,
        "t": 10.00997909802668,
        "x": 294.8666745880158,
        "y": 0,
        "z": 228.5789620872062,
        "shotId": 82
      },
      {
        "id": 16022,
        "t": 10.04059125162456,
        "x": 294.9184374606442,
        "y": 0,
        "z": 228.6248236907755,
        "shotId": 82
      },
      {
        "id": 16023,
        "t": 10.07120340522244,
        "x": 294.969214373794,
        "y": 0,
        "z": 228.6698117399912,
        "shotId": 82
      },
      {
        "id": 16024,
        "t": 10.10181555882031,
        "x": 295.0190053274652,
        "y": 0,
        "z": 228.7139262348532,
        "shotId": 82
      },
      {
        "id": 16025,
        "t": 10.13242771241819,
        "x": 295.0678103216576,
        "y": 0,
        "z": 228.7571671753615,
        "shotId": 82
      },
      {
        "id": 16026,
        "t": 10.16303986601607,
        "x": 295.1156293563716,
        "y": 0,
        "z": 228.7995345615161,
        "shotId": 82
      },
      {
        "id": 16027,
        "t": 10.19365201961395,
        "x": 295.1624624316069,
        "y": 0,
        "z": 228.841028393317,
        "shotId": 82
      },
      {
        "id": 16028,
        "t": 10.22426417321182,
        "x": 295.2083095473635,
        "y": 0,
        "z": 228.8816486707642,
        "shotId": 82
      },
      {
        "id": 16029,
        "t": 10.2548763268097,
        "x": 295.2531707036414,
        "y": 0,
        "z": 228.9213953938577,
        "shotId": 82
      },
      {
        "id": 16030,
        "t": 10.28548848040758,
        "x": 295.2970459004408,
        "y": 0,
        "z": 228.9602685625975,
        "shotId": 82
      },
      {
        "id": 16031,
        "t": 10.31610063400545,
        "x": 295.3399351377615,
        "y": 0,
        "z": 228.9982681769836,
        "shotId": 82
      },
      {
        "id": 16032,
        "t": 10.34671278760333,
        "x": 295.3818384156036,
        "y": 0,
        "z": 229.0353942370159,
        "shotId": 82
      },
      {
        "id": 16033,
        "t": 10.37732494120121,
        "x": 295.422755733967,
        "y": 0,
        "z": 229.0716467426946,
        "shotId": 82
      },
      {
        "id": 16034,
        "t": 10.40793709479909,
        "x": 295.4626870928518,
        "y": 0,
        "z": 229.1070256940196,
        "shotId": 82
      },
      {
        "id": 16035,
        "t": 10.43854924839696,
        "x": 295.5016324922579,
        "y": 0,
        "z": 229.1415310909908,
        "shotId": 82
      },
      {
        "id": 16036,
        "t": 10.46916140199484,
        "x": 295.5395919321854,
        "y": 0,
        "z": 229.1751629336084,
        "shotId": 82
      },
      {
        "id": 16037,
        "t": 10.49977355559272,
        "x": 295.5765654126344,
        "y": 0,
        "z": 229.2079212218723,
        "shotId": 82
      },
      {
        "id": 16038,
        "t": 10.5303857091906,
        "x": 295.6125529336046,
        "y": 0,
        "z": 229.2398059557824,
        "shotId": 82
      },
      {
        "id": 16039,
        "t": 10.56099786278847,
        "x": 295.6475544950962,
        "y": 0,
        "z": 229.2708171353389,
        "shotId": 82
      },
      {
        "id": 16040,
        "t": 10.59161001638635,
        "x": 295.6815700971092,
        "y": 0,
        "z": 229.3009547605416,
        "shotId": 82
      },
      {
        "id": 16041,
        "t": 10.62222216998423,
        "x": 295.7145997396435,
        "y": 0,
        "z": 229.3302188313907,
        "shotId": 82
      },
      {
        "id": 16042,
        "t": 10.6528343235821,
        "x": 295.7466434226992,
        "y": 0,
        "z": 229.358609347886,
        "shotId": 82
      },
      {
        "id": 16043,
        "t": 10.68344647717998,
        "x": 295.7777011462763,
        "y": 0,
        "z": 229.3861263100277,
        "shotId": 82
      },
      {
        "id": 16044,
        "t": 10.71405863077786,
        "x": 295.8077729103747,
        "y": 0,
        "z": 229.4127697178156,
        "shotId": 82
      },
      {
        "id": 16045,
        "t": 10.74467078437574,
        "x": 295.8368587149945,
        "y": 0,
        "z": 229.4385395712499,
        "shotId": 82
      },
      {
        "id": 16046,
        "t": 10.77528293797361,
        "x": 295.8649585601357,
        "y": 0,
        "z": 229.4634358703304,
        "shotId": 82
      },
      {
        "id": 16047,
        "t": 10.80589509157149,
        "x": 295.8920724457982,
        "y": 0,
        "z": 229.4874586150572,
        "shotId": 82
      },
      {
        "id": 16048,
        "t": 10.83650724516936,
        "x": 295.918200371982,
        "y": 0,
        "z": 229.5106078054303,
        "shotId": 82
      },
      {
        "id": 16049,
        "t": 10.86711939876724,
        "x": 295.9433423386873,
        "y": 0,
        "z": 229.5328834414498,
        "shotId": 82
      },
      {
        "id": 16050,
        "t": 10.89773155236512,
        "x": 295.967498345914,
        "y": 0,
        "z": 229.5542855231155,
        "shotId": 82
      },
      {
        "id": 16051,
        "t": 10.928343705963,
        "x": 295.9906683936618,
        "y": 0,
        "z": 229.5748140504275,
        "shotId": 82
      },
      {
        "id": 16052,
        "t": 10.95895585956087,
        "x": 296.0128524819312,
        "y": 0,
        "z": 229.5944690233858,
        "shotId": 82
      },
      {
        "id": 16053,
        "t": 10.98956801315875,
        "x": 296.0340506107219,
        "y": 0,
        "z": 229.6132504419904,
        "shotId": 82
      },
      {
        "id": 16054,
        "t": 11.02018016675663,
        "x": 296.054262780034,
        "y": 0,
        "z": 229.6311583062414,
        "shotId": 82
      },
      {
        "id": 16055,
        "t": 11.05079232035451,
        "x": 296.0734889898674,
        "y": 0,
        "z": 229.6481926161385,
        "shotId": 82
      },
      {
        "id": 16056,
        "t": 11.08140447395238,
        "x": 296.0917292402222,
        "y": 0,
        "z": 229.6643533716821,
        "shotId": 82
      },
      {
        "id": 16057,
        "t": 11.11201662755026,
        "x": 296.1089835310983,
        "y": 0,
        "z": 229.6796405728718,
        "shotId": 82
      },
      {
        "id": 16058,
        "t": 11.14262878114814,
        "x": 296.1252518624958,
        "y": 0,
        "z": 229.694054219708,
        "shotId": 82
      },
      {
        "id": 16059,
        "t": 11.17324093474601,
        "x": 296.1405342344147,
        "y": 0,
        "z": 229.7075943121903,
        "shotId": 82
      },
      {
        "id": 16060,
        "t": 11.20385308834389,
        "x": 296.1548306468549,
        "y": 0,
        "z": 229.7202608503191,
        "shotId": 82
      },
      {
        "id": 16061,
        "t": 11.23446524194177,
        "x": 296.1681410998165,
        "y": 0,
        "z": 229.732053834094,
        "shotId": 82
      },
      {
        "id": 16062,
        "t": 11.26507739553965,
        "x": 296.1804655932995,
        "y": 0,
        "z": 229.7429732635153,
        "shotId": 82
      },
      {
        "id": 16063,
        "t": 11.29568954913752,
        "x": 296.1918041273038,
        "y": 0,
        "z": 229.7530191385829,
        "shotId": 82
      },
      {
        "id": 16064,
        "t": 11.3263017027354,
        "x": 296.2021567018295,
        "y": 0,
        "z": 229.7621914592968,
        "shotId": 82
      },
      {
        "id": 16065,
        "t": 11.35691385633328,
        "x": 296.2115233168765,
        "y": 0,
        "z": 229.770490225657,
        "shotId": 82
      },
      {
        "id": 16066,
        "t": 11.38752600993115,
        "x": 296.219903972445,
        "y": 0,
        "z": 229.7779154376634,
        "shotId": 82
      },
      {
        "id": 16067,
        "t": 11.41813816352903,
        "x": 296.2272986685347,
        "y": 0,
        "z": 229.7844670953162,
        "shotId": 82
      },
      {
        "id": 16068,
        "t": 11.44875031712691,
        "x": 296.2337074051459,
        "y": 0,
        "z": 229.7901451986153,
        "shotId": 82
      },
      {
        "id": 16069,
        "t": 11.47936247072479,
        "x": 296.2391301822784,
        "y": 0,
        "z": 229.7949497475607,
        "shotId": 82
      },
      {
        "id": 16070,
        "t": 11.50997462432266,
        "x": 296.2435669999322,
        "y": 0,
        "z": 229.7988807421523,
        "shotId": 82
      },
      {
        "id": 16071,
        "t": 11.54058677792054,
        "x": 296.2470178581074,
        "y": 0,
        "z": 229.8019381823903,
        "shotId": 82
      },
      {
        "id": 16072,
        "t": 11.57119893151842,
        "x": 296.2494827568041,
        "y": 0,
        "z": 229.8041220682746,
        "shotId": 82
      },
      {
        "id": 16073,
        "t": 11.60181108511629,
        "x": 296.250961696022,
        "y": 0,
        "z": 229.8054323998051,
        "shotId": 82
      },
      {
        "id": 16074,
        "t": 11.63242323871417,
        "x": 296.2514546757613,
        "y": 0,
        "z": 229.8058691769819,
        "shotId": 82
      }
    ]
  },
  {
    "id": 83,
    "number": 2,
    "ballMph": 180,
    "launchAngle": 22,
    "verticalRpm": 2222,
    "horizontalRpm": -800,
    "carryYards": null,
    "totalYards": null,
    "landingAngle": 0,
    "deviationAngle": -55,
    "smashFactor": null,
    "ignore": false,
    "manuallyEntered": true,
    "rdPeakHeightYards": 47.86307895081447,
    "rdCarryYards": 272.843176550648,
    "rdRollYards": 284.8887962068758,
    "rdDevCarryYards": -250.1165329460069,
    "rdDevRollYards": -261.3160437673297,
    "rdLandingDeg": -46.86189546728548,
    "carryIdx": 66,
    "rollIdx": 264,
    "clubConfigId": 54,
    "average": false,
    "shotGraphs": [
      {
        "id": 16075,
        "t": 0,
        "x": 0,
        "y": 0,
        "z": 0,
        "shotId": 83
      },
      {
        "id": 16076,
        "t": 0.1,
        "x": 11.43390013172545,
        "y": 3.23393606090746,
        "z": -6.59348337441446,
        "shotId": 83
      },
      {
        "id": 16077,
        "t": 0.2,
        "x": 22.26576355535991,
        "y": 6.343343596793749,
        "z": -13.01333278372124,
        "shotId": 83
      },
      {
        "id": 16078,
        "t": 0.3,
        "x": 32.53715885141455,
        "y": 9.329346768569929,
        "z": -19.26893217431092,
        "shotId": 83
      },
      {
        "id": 16079,
        "t": 0.4,
        "x": 42.28605666844309,
        "y": 12.19314954076195,
        "z": -25.36892170142612,
        "shotId": 83
      },
      {
        "id": 16080,
        "t": 0.5,
        "x": 51.5472622112715,
        "y": 14.93601472743716,
        "z": -31.32128635563741,
        "shotId": 83
      },
      {
        "id": 16081,
        "t": 0.6,
        "x": 60.35290502087029,
        "y": 17.55917505000187,
        "z": -37.13337212862341,
        "shotId": 83
      },
      {
        "id": 16082,
        "t": 0.7,
        "x": 68.73345349155963,
        "y": 20.06377191246305,
        "z": -42.81207656756548,
        "shotId": 83
      },
      {
        "id": 16083,
        "t": 0.7999999999999999,
        "x": 76.71773010359372,
        "y": 22.45093922921726,
        "z": -48.36395676918963,
        "shotId": 83
      },
      {
        "id": 16084,
        "t": 0.8999999999999999,
        "x": 84.3324820059251,
        "y": 24.72186247239118,
        "z": -53.79518853334216,
        "shotId": 83
      },
      {
        "id": 16085,
        "t": 0.9999999999999999,
        "x": 91.6020546215695,
        "y": 26.87780270522002,
        "z": -59.11150346613185,
        "shotId": 83
      },
      {
        "id": 16086,
        "t": 1.1,
        "x": 98.54880865835847,
        "y": 28.92005744264041,
        "z": -64.31825379770429,
        "shotId": 83
      },
      {
        "id": 16087,
        "t": 1.2,
        "x": 105.1934697670874,
        "y": 30.84993669161608,
        "z": -69.42047786878315,
        "shotId": 83
      },
      {
        "id": 16088,
        "t": 1.3,
        "x": 111.5552302849149,
        "y": 32.66875689155723,
        "z": -74.422916657196,
        "shotId": 83
      },
      {
        "id": 16089,
        "t": 1.4,
        "x": 117.6517881113154,
        "y": 34.37783295103623,
        "z": -79.33000419448145,
        "shotId": 83
      },
      {
        "id": 16090,
        "t": 1.5,
        "x": 123.4992533731549,
        "y": 35.97846677312026,
        "z": -84.14580123887728,
        "shotId": 83
      },
      {
        "id": 16091,
        "t": 1.6,
        "x": 129.1119599879906,
        "y": 37.47197142212315,
        "z": -88.87393941732186,
        "shotId": 83
      },
      {
        "id": 16092,
        "t": 1.7,
        "x": 134.5024931023136,
        "y": 38.85967839134546,
        "z": -93.51760781685068,
        "shotId": 83
      },
      {
        "id": 16093,
        "t": 1.8,
        "x": 139.6820704801699,
        "y": 40.14291883961185,
        "z": -98.07962758898235,
        "shotId": 83
      },
      {
        "id": 16094,
        "t": 1.900000000000001,
        "x": 144.6609055788296,
        "y": 41.32300687745726,
        "z": -102.5625305190789,
        "shotId": 83
      },
      {
        "id": 16095,
        "t": 2,
        "x": 149.4485050707699,
        "y": 42.40122558205273,
        "z": -106.9686290761287,
        "shotId": 83
      },
      {
        "id": 16096,
        "t": 2.100000000000001,
        "x": 154.0538442459205,
        "y": 43.37882099808976,
        "z": -111.3000608616696,
        "shotId": 83
      },
      {
        "id": 16097,
        "t": 2.200000000000001,
        "x": 158.4854974614772,
        "y": 44.25699627800239,
        "z": -115.5588159606537,
        "shotId": 83
      },
      {
        "id": 16098,
        "t": 2.300000000000001,
        "x": 162.7517286280872,
        "y": 45.03690920886485,
        "z": -119.7467626810372,
        "shotId": 83
      },
      {
        "id": 16099,
        "t": 2.400000000000001,
        "x": 166.8605912564681,
        "y": 45.71966795301194,
        "z": -123.8656781401262,
        "shotId": 83
      },
      {
        "id": 16100,
        "t": 2.500000000000001,
        "x": 170.8199949362063,
        "y": 46.30633504695537,
        "z": -127.9172896895665,
        "shotId": 83
      },
      {
        "id": 16101,
        "t": 2.600000000000001,
        "x": 174.6376869910997,
        "y": 46.79792867610241,
        "z": -131.903264533425,
        "shotId": 83
      },
      {
        "id": 16102,
        "t": 2.700000000000001,
        "x": 178.3212498872999,
        "y": 47.19543243353569,
        "z": -135.8252315428048,
        "shotId": 83
      },
      {
        "id": 16103,
        "t": 2.800000000000001,
        "x": 181.8781019389211,
        "y": 47.4997910113613,
        "z": -139.6847709959213,
        "shotId": 83
      },
      {
        "id": 16104,
        "t": 2.900000000000001,
        "x": 185.315450852221,
        "y": 47.7119267948514,
        "z": -143.4834247440117,
        "shotId": 83
      },
      {
        "id": 16105,
        "t": 3.000000000000001,
        "x": 188.6403167587251,
        "y": 47.8327316856836,
        "z": -147.2226933607492,
        "shotId": 83
      },
      {
        "id": 16106,
        "t": 3.100000000000001,
        "x": 191.8594914379545,
        "y": 47.86307895081447,
        "z": -150.9040336415468,
        "shotId": 83
      },
      {
        "id": 16107,
        "t": 3.200000000000002,
        "x": 194.9795148278471,
        "y": 47.80383103374957,
        "z": -154.5288576602505,
        "shotId": 83
      },
      {
        "id": 16108,
        "t": 3.300000000000002,
        "x": 198.0067151526594,
        "y": 47.65582664734152,
        "z": -158.0985311161012,
        "shotId": 83
      },
      {
        "id": 16109,
        "t": 3.400000000000002,
        "x": 200.9471530916434,
        "y": 47.41990151777566,
        "z": -161.6143703760842,
        "shotId": 83
      },
      {
        "id": 16110,
        "t": 3.500000000000002,
        "x": 203.8066156419897,
        "y": 47.09689313746971,
        "z": -165.077636940987,
        "shotId": 83
      },
      {
        "id": 16111,
        "t": 3.600000000000002,
        "x": 206.5906308812604,
        "y": 46.68763389327026,
        "z": -168.4895151538114,
        "shotId": 83
      },
      {
        "id": 16112,
        "t": 3.700000000000002,
        "x": 209.3044738810268,
        "y": 46.19295873728829,
        "z": -171.8511287777441,
        "shotId": 83
      },
      {
        "id": 16113,
        "t": 3.800000000000002,
        "x": 211.9531757510743,
        "y": 45.61371167177825,
        "z": -175.1635576312885,
        "shotId": 83
      },
      {
        "id": 16114,
        "t": 3.900000000000002,
        "x": 214.5415515442197,
        "y": 44.9507381550268,
        "z": -178.4278434458075,
        "shotId": 83
      },
      {
        "id": 16115,
        "t": 4.000000000000002,
        "x": 217.0742006567111,
        "y": 44.20488767278262,
        "z": -181.6449775962886,
        "shotId": 83
      },
      {
        "id": 16116,
        "t": 4.100000000000001,
        "x": 219.5555004339225,
        "y": 43.37702331609004,
        "z": -184.815892094958,
        "shotId": 83
      },
      {
        "id": 16117,
        "t": 4.200000000000001,
        "x": 221.9896295217777,
        "y": 42.46802174758677,
        "z": -187.941477575726,
        "shotId": 83
      },
      {
        "id": 16118,
        "t": 4.300000000000001,
        "x": 224.3805829049349,
        "y": 41.47877314710524,
        "z": -191.0225864695702,
        "shotId": 83
      },
      {
        "id": 16119,
        "t": 4.4,
        "x": 226.7321850759159,
        "y": 40.41018129332996,
        "z": -194.0600343640727,
        "shotId": 83
      },
      {
        "id": 16120,
        "t": 4.5,
        "x": 229.0481024133546,
        "y": 39.26316369150113,
        "z": -197.0546015396477,
        "shotId": 83
      },
      {
        "id": 16121,
        "t": 4.6,
        "x": 231.3318547581731,
        "y": 38.03865171346068,
        "z": -200.0070346481877,
        "shotId": 83
      },
      {
        "id": 16122,
        "t": 4.699999999999999,
        "x": 233.5868261832558,
        "y": 36.73759072147197,
        "z": -202.918048502162,
        "shotId": 83
      },
      {
        "id": 16123,
        "t": 4.799999999999999,
        "x": 235.8162749586518,
        "y": 35.36094015217785,
        "z": -205.7883279444407,
        "shotId": 83
      },
      {
        "id": 16124,
        "t": 4.899999999999999,
        "x": 238.0233427205137,
        "y": 33.90967354175078,
        "z": -208.618529771333,
        "shotId": 83
      },
      {
        "id": 16125,
        "t": 4.999999999999998,
        "x": 240.2110628578345,
        "y": 32.38477847768765,
        "z": -211.4092846835559,
        "shotId": 83
      },
      {
        "id": 16126,
        "t": 5.099999999999998,
        "x": 242.3823681365401,
        "y": 30.78725646677274,
        "z": -214.1611992421061,
        "shotId": 83
      },
      {
        "id": 16127,
        "t": 5.199999999999998,
        "x": 244.5400975855459,
        "y": 29.11812271243794,
        "z": -216.8748578082852,
        "shotId": 83
      },
      {
        "id": 16128,
        "t": 5.299999999999997,
        "x": 246.6870026739387,
        "y": 27.37840579806546,
        "z": -219.5508244494371,
        "shotId": 83
      },
      {
        "id": 16129,
        "t": 5.399999999999997,
        "x": 248.8257528124296,
        "y": 25.56914727568591,
        "z": -222.1896447942538,
        "shotId": 83
      },
      {
        "id": 16130,
        "t": 5.499999999999996,
        "x": 250.9589402155886,
        "y": 23.69140116201962,
        "z": -224.7918478237917,
        "shotId": 83
      },
      {
        "id": 16131,
        "t": 5.599999999999996,
        "x": 253.0890841640827,
        "y": 21.74623334589262,
        "z": -227.3579475865688,
        "shotId": 83
      },
      {
        "id": 16132,
        "t": 5.699999999999996,
        "x": 255.218634708186,
        "y": 19.73472091274539,
        "z": -229.8884448282711,
        "shotId": 83
      },
      {
        "id": 16133,
        "t": 5.799999999999995,
        "x": 257.349975855206,
        "y": 17.65795139326356,
        "z": -232.3838285286484,
        "shotId": 83
      },
      {
        "id": 16134,
        "t": 5.899999999999995,
        "x": 259.485428284205,
        "y": 15.51702194412132,
        "z": -234.8445773401064,
        "shotId": 83
      },
      {
        "id": 16135,
        "t": 5.999999999999995,
        "x": 261.6272516315238,
        "y": 13.31303846947514,
        "z": -237.2711609242877,
        "shotId": 83
      },
      {
        "id": 16136,
        "t": 6.099999999999994,
        "x": 263.7776463901808,
        "y": 11.04711469221188,
        "z": -239.6640411845567,
        "shotId": 83
      },
      {
        "id": 16137,
        "t": 6.199999999999994,
        "x": 265.9387554652962,
        "y": 8.720371184078783,
        "z": -242.0236733937643,
        "shotId": 83
      },
      {
        "id": 16138,
        "t": 6.299999999999994,
        "x": 268.1126654263242,
        "y": 6.33393436374149,
        "z": -244.3505072179524,
        "shotId": 83
      },
      {
        "id": 16139,
        "t": 6.399999999999993,
        "x": 270.3014074951583,
        "y": 3.888935471564835,
        "z": -246.6449876377741,
        "shotId": 83
      },
      {
        "id": 16140,
        "t": 6.499999999999993,
        "x": 272.5069560270914,
        "y": 1.386510747349009,
        "z": -248.9075544001548,
        "shotId": 83
      },
      {
        "id": 16141,
        "t": 6.554188016356292,
        "x": 273.7122385236627,
        "y": -2.428279799460143e-16,
        "z": -250.1165329460069,
        "shotId": 83
      },
      {
        "id": 16142,
        "t": 6.554188016356292,
        "x": 273.7122385236627,
        "y": -2.428279799460143e-16,
        "z": -250.1165329460069,
        "shotId": 83
      },
      {
        "id": 16143,
        "t": 6.574188016356292,
        "x": 273.7972113515874,
        "y": 0.1197650377372525,
        "z": -250.201680281424,
        "shotId": 83
      },
      {
        "id": 16144,
        "t": 6.594188016356292,
        "x": 273.8821825748194,
        "y": 0.2351510896220802,
        "z": -250.2868260088528,
        "shotId": 83
      },
      {
        "id": 16145,
        "t": 6.614188016356292,
        "x": 273.9671507320318,
        "y": 0.3461624728336565,
        "z": -250.3719686639655,
        "shotId": 83
      },
      {
        "id": 16146,
        "t": 6.634188016356292,
        "x": 274.0521144328453,
        "y": 0.4528033689132752,
        "z": -250.4571068535272,
        "shotId": 83
      },
      {
        "id": 16147,
        "t": 6.654188016356292,
        "x": 274.1370723557807,
        "y": 0.5550778242668423,
        "z": -250.5422392533448,
        "shotId": 83
      },
      {
        "id": 16148,
        "t": 6.674188016356292,
        "x": 274.2220232460673,
        "y": 0.6529897507036381,
        "z": -250.6273646060708,
        "shotId": 83
      },
      {
        "id": 16149,
        "t": 6.694188016356292,
        "x": 274.3069659132892,
        "y": 0.7465429260220063,
        "z": -250.7124817188445,
        "shotId": 83
      },
      {
        "id": 16150,
        "t": 6.714188016356292,
        "x": 274.3918992288542,
        "y": 0.8357409946545695,
        "z": -250.7975894607559,
        "shotId": 83
      },
      {
        "id": 16151,
        "t": 6.734188016356292,
        "x": 274.4768221232663,
        "y": 0.9205874683877978,
        "z": -250.8826867601127,
        "shotId": 83
      },
      {
        "id": 16152,
        "t": 6.754188016356292,
        "x": 274.5617335831842,
        "y": 1.001085727173287,
        "z": -250.9677726014923,
        "shotId": 83
      },
      {
        "id": 16153,
        "t": 6.774188016356292,
        "x": 274.6466326482453,
        "y": 1.077239020050939,
        "z": -251.0528460225601,
        "shotId": 83
      },
      {
        "id": 16154,
        "t": 6.794188016356292,
        "x": 274.7315184076374,
        "y": 1.149050466207384,
        "z": -251.1379061106332,
        "shotId": 83
      },
      {
        "id": 16155,
        "t": 6.814188016356292,
        "x": 274.816389996397,
        "y": 1.216523056196359,
        "z": -251.2229519989717,
        "shotId": 83
      },
      {
        "id": 16156,
        "t": 6.834188016356292,
        "x": 274.9012465914174,
        "y": 1.279659653351382,
        "z": -251.3079828627787,
        "shotId": 83
      },
      {
        "id": 16157,
        "t": 6.854188016356292,
        "x": 274.9860874071504,
        "y": 1.338462995424717,
        "z": -251.3929979148925,
        "shotId": 83
      },
      {
        "id": 16158,
        "t": 6.874188016356293,
        "x": 275.0709116909864,
        "y": 1.392935696490236,
        "z": -251.477996401158,
        "shotId": 83
      },
      {
        "id": 16159,
        "t": 6.894188016356292,
        "x": 275.1557187183052,
        "y": 1.443080249151067,
        "z": -251.5629775954668,
        "shotId": 83
      },
      {
        "id": 16160,
        "t": 6.914188016356293,
        "x": 275.240507787192,
        "y": 1.488899027095636,
        "z": -251.6479407944627,
        "shotId": 83
      },
      {
        "id": 16161,
        "t": 6.934188016356292,
        "x": 275.3252782128221,
        "y": 1.530394288047405,
        "z": -251.7328853119144,
        "shotId": 83
      },
      {
        "id": 16162,
        "t": 6.954188016356293,
        "x": 275.410029321524,
        "y": 1.567568177153995,
        "z": -251.8178104727671,
        "shotId": 83
      },
      {
        "id": 16163,
        "t": 6.974188016356292,
        "x": 275.4947604445435,
        "y": 1.600422730859857,
        "z": -251.9027156068931,
        "shotId": 83
      },
      {
        "id": 16164,
        "t": 6.994188016356293,
        "x": 275.5794709115385,
        "y": 1.628959881302872,
        "z": -251.9876000425736,
        "shotId": 83
      },
      {
        "id": 16165,
        "t": 7.014188016356292,
        "x": 275.664160043849,
        "y": 1.653181461268816,
        "z": -252.0724630997548,
        "shotId": 83
      },
      {
        "id": 16166,
        "t": 7.034188016356293,
        "x": 275.7488271475972,
        "y": 1.673089209728195,
        "z": -252.157304083134,
        "shotId": 83
      },
      {
        "id": 16167,
        "t": 7.054188016356292,
        "x": 275.8334715066839,
        "y": 1.688684777967626,
        "z": -252.2421222751413,
        "shotId": 83
      },
      {
        "id": 16168,
        "t": 7.074188016356293,
        "x": 275.9180923757573,
        "y": 1.699969736312851,
        "z": -252.3269169288943,
        "shotId": 83
      },
      {
        "id": 16169,
        "t": 7.094188016356292,
        "x": 276.002688973238,
        "y": 1.706945581423229,
        "z": -252.4116872612081,
        "shotId": 83
      },
      {
        "id": 16170,
        "t": 7.114188016356293,
        "x": 276.0872604744855,
        "y": 1.709613744119127,
        "z": -252.4964324457492,
        "shotId": 83
      },
      {
        "id": 16171,
        "t": 7.134188016356292,
        "x": 276.1718060051968,
        "y": 1.707975597685065,
        "z": -252.5811516064186,
        "shotId": 83
      },
      {
        "id": 16172,
        "t": 7.154188016356293,
        "x": 276.2563246351157,
        "y": 1.702032466574313,
        "z": -252.6658438110499,
        "shotId": 83
      },
      {
        "id": 16173,
        "t": 7.174188016356292,
        "x": 276.3408153721306,
        "y": 1.691785635425947,
        "z": -252.750508065494,
        "shotId": 83
      },
      {
        "id": 16174,
        "t": 7.194188016356293,
        "x": 276.4252771568219,
        "y": 1.677236358294672,
        "z": -252.8351433081554,
        "shotId": 83
      },
      {
        "id": 16175,
        "t": 7.214188016356292,
        "x": 276.5097088575061,
        "y": 1.658385867987478,
        "z": -252.9197484050266,
        "shotId": 83
      },
      {
        "id": 16176,
        "t": 7.234188016356293,
        "x": 276.5941092658095,
        "y": 1.635235385400055,
        "z": -253.0043221452524,
        "shotId": 83
      },
      {
        "id": 16177,
        "t": 7.254188016356292,
        "x": 276.6784770927853,
        "y": 1.607786128749648,
        "z": -253.0888632372387,
        "shotId": 83
      },
      {
        "id": 16178,
        "t": 7.274188016356293,
        "x": 276.7628109655728,
        "y": 1.576039322609031,
        "z": -253.1733703053055,
        "shotId": 83
      },
      {
        "id": 16179,
        "t": 7.294188016356292,
        "x": 276.8471094245849,
        "y": 1.539996206657817,
        "z": -253.2578418868681,
        "shotId": 83
      },
      {
        "id": 16180,
        "t": 7.314188016356293,
        "x": 276.9313709211957,
        "y": 1.499658044081089,
        "z": -253.3422764301202,
        "shotId": 83
      },
      {
        "id": 16181,
        "t": 7.334188016356292,
        "x": 277.0155938158937,
        "y": 1.455026129560314,
        "z": -253.4266722921834,
        "shotId": 83
      },
      {
        "id": 16182,
        "t": 7.354188016356293,
        "x": 277.0997763768597,
        "y": 1.406101796816629,
        "z": -253.5110277376818,
        "shotId": 83
      },
      {
        "id": 16183,
        "t": 7.374188016356293,
        "x": 277.1839167789233,
        "y": 1.352886425680909,
        "z": -253.5953409376966,
        "shotId": 83
      },
      {
        "id": 16184,
        "t": 7.394188016356293,
        "x": 277.2680131028539,
        "y": 1.295381448677986,
        "z": -253.6796099690557,
        "shotId": 83
      },
      {
        "id": 16185,
        "t": 7.414188016356293,
        "x": 277.3520633349416,
        "y": 1.233588357123572,
        "z": -253.7638328139139,
        "shotId": 83
      },
      {
        "id": 16186,
        "t": 7.434188016356293,
        "x": 277.4360653668253,
        "y": 1.167508706741595,
        "z": -253.8480073595798,
        "shotId": 83
      },
      {
        "id": 16187,
        "t": 7.454188016356293,
        "x": 277.5200169955285,
        "y": 1.097144122816877,
        "z": -253.9321313985531,
        "shotId": 83
      },
      {
        "id": 16188,
        "t": 7.474188016356293,
        "x": 277.6039159236701,
        "y": 1.022496304903412,
        "z": -254.0162026287343,
        "shotId": 83
      },
      {
        "id": 16189,
        "t": 7.494188016356293,
        "x": 277.6877597598166,
        "y": 0.9435670311121958,
        "z": -254.1002186537788,
        "shotId": 83
      },
      {
        "id": 16190,
        "t": 7.514188016356293,
        "x": 277.7715460189498,
        "y": 0.8603581620048069,
        "z": -254.1841769835649,
        "shotId": 83
      },
      {
        "id": 16191,
        "t": 7.534188016356293,
        "x": 277.8552721230275,
        "y": 0.7728716441200624,
        "z": -254.2680750347559,
        "shotId": 83
      },
      {
        "id": 16192,
        "t": 7.554188016356292,
        "x": 277.9389354016171,
        "y": 0.6811095131612634,
        "z": -254.3519101314349,
        "shotId": 83
      },
      {
        "id": 16193,
        "t": 7.574188016356293,
        "x": 278.0225330925852,
        "y": 0.585073896871053,
        "z": -254.4356795057962,
        "shotId": 83
      },
      {
        "id": 16194,
        "t": 7.594188016356293,
        "x": 278.106062342832,
        "y": 0.4847670176199176,
        "z": -254.5193802988803,
        "shotId": 83
      },
      {
        "id": 16195,
        "t": 7.614188016356293,
        "x": 278.189520209057,
        "y": 0.3801911947330362,
        "z": -254.6030095613423,
        "shotId": 83
      },
      {
        "id": 16196,
        "t": 7.634188016356292,
        "x": 278.2729036585489,
        "y": 0.2713488465786388,
        "z": -254.6865642542427,
        "shotId": 83
      },
      {
        "id": 16197,
        "t": 7.654188016356293,
        "x": 278.3562095699926,
        "y": 0.1582424924393856,
        "z": -254.770041249856,
        "shotId": 83
      },
      {
        "id": 16198,
        "t": 7.674188016356293,
        "x": 278.4394347342869,
        "y": 0.04087475418658908,
        "z": -254.8534373324908,
        "shotId": 83
      },
      {
        "id": 16199,
        "t": 7.680909378723786,
        "x": 278.4673758144089,
        "y": -2.428279799460143e-16,
        "z": -254.881435794813,
        "shotId": 83
      },
      {
        "id": 16200,
        "t": 7.680909378723786,
        "x": 278.4673758144089,
        "y": -2.428279799460143e-16,
        "z": -254.881435794813,
        "shotId": 83
      },
      {
        "id": 16201,
        "t": 7.700909378723786,
        "x": 278.5411430648772,
        "y": 0.0424682001275747,
        "z": -254.9553545400381,
        "shotId": 83
      },
      {
        "id": 16202,
        "t": 7.720909378723786,
        "x": 278.6148965277281,
        "y": 0.0806211873189831,
        "z": -255.0292594693304,
        "shotId": 83
      },
      {
        "id": 16203,
        "t": 7.740909378723786,
        "x": 278.6886356113048,
        "y": 0.1144608742596345,
        "z": -255.103149989818,
        "shotId": 83
      },
      {
        "id": 16204,
        "t": 7.760909378723786,
        "x": 278.7623597315704,
        "y": 0.1439890638001337,
        "z": -255.1770255162645,
        "shotId": 83
      },
      {
        "id": 16205,
        "t": 7.780909378723786,
        "x": 278.8360683047879,
        "y": 0.1692074536956535,
        "z": -255.2508854637341,
        "shotId": 83
      },
      {
        "id": 16206,
        "t": 7.800909378723786,
        "x": 278.90976073988,
        "y": 0.19011764206956,
        "z": -255.3247292399357,
        "shotId": 83
      },
      {
        "id": 16207,
        "t": 7.820909378723786,
        "x": 278.9834364305478,
        "y": 0.2067211336418648,
        "z": -255.3985562373251,
        "shotId": 83
      },
      {
        "id": 16208,
        "t": 7.840909378723786,
        "x": 279.0570947472451,
        "y": 0.2190193467425209,
        "z": -255.4723658250635,
        "shotId": 83
      },
      {
        "id": 16209,
        "t": 7.860909378723786,
        "x": 279.1307350291266,
        "y": 0.2270136211036066,
        "z": -255.5461573409483,
        "shotId": 83
      },
      {
        "id": 16210,
        "t": 7.880909378723786,
        "x": 279.2043565760942,
        "y": 0.2307052263943519,
        "z": -255.6199300834434,
        "shotId": 83
      },
      {
        "id": 16211,
        "t": 7.900909378723786,
        "x": 279.2779586410749,
        "y": 0.230095371430902,
        "z": -255.6936833039418,
        "shotId": 83
      },
      {
        "id": 16212,
        "t": 7.920909378723787,
        "x": 279.3515404226591,
        "y": 0.2251852139614746,
        "z": -255.767416199388,
        "shotId": 83
      },
      {
        "id": 16213,
        "t": 7.940909378723786,
        "x": 279.4251010582158,
        "y": 0.215975870900229,
        "z": -255.8411279053794,
        "shotId": 83
      },
      {
        "id": 16214,
        "t": 7.960909378723787,
        "x": 279.498639617584,
        "y": 0.2024684288625592,
        "z": -255.9148174898446,
        "shotId": 83
      },
      {
        "id": 16215,
        "t": 7.980909378723786,
        "x": 279.5721550974119,
        "y": 0.1846639548427681,
        "z": -255.9884839473716,
        "shotId": 83
      },
      {
        "id": 16216,
        "t": 8.000909378723787,
        "x": 279.6456464161922,
        "y": 0.1625635068731562,
        "z": -256.0621261942316,
        "shotId": 83
      },
      {
        "id": 16217,
        "t": 8.020909378723786,
        "x": 279.7191124100069,
        "y": 0.1361681445112129,
        "z": -256.1357430641166,
        "shotId": 83
      },
      {
        "id": 16218,
        "t": 8.040909378723786,
        "x": 279.7925518289747,
        "y": 0.1054789390174254,
        "z": -256.2093333045783,
        "shotId": 83
      },
      {
        "id": 16219,
        "t": 8.060909378723787,
        "x": 279.865963334367,
        "y": 0.07049698310802234,
        "z": -256.2828955741388,
        "shotId": 83
      },
      {
        "id": 16220,
        "t": 8.080909378723787,
        "x": 279.9393454963443,
        "y": 0.03122340019217879,
        "z": -256.3564284400222,
        "shotId": 83
      },
      {
        "id": 16221,
        "t": 8.09524385966651,
        "x": 279.9919181340099,
        "y": -2.428279799460143e-16,
        "z": -256.4091090453686,
        "shotId": 83
      },
      {
        "id": 16222,
        "t": 8.09524385966651,
        "x": 279.9919181340099,
        "y": -2.428279799460143e-16,
        "z": -256.4091090453686,
        "shotId": 83
      },
      {
        "id": 16223,
        "t": 8.11524385966651,
        "x": 280.0583218837486,
        "y": 0.01853301722324284,
        "z": -256.475649167547,
        "shotId": 83
      },
      {
        "id": 16224,
        "t": 8.13524385966651,
        "x": 280.1247119686877,
        "y": 0.0327635621642439,
        "z": -256.5421755968626,
        "shotId": 83
      },
      {
        "id": 16225,
        "t": 8.15524385966651,
        "x": 280.1910878156098,
        "y": 0.04269289668120986,
        "z": -256.6086877589208,
        "shotId": 83
      },
      {
        "id": 16226,
        "t": 8.17524385966651,
        "x": 280.2574488179463,
        "y": 0.04832220632521555,
        "z": -256.6751850459073,
        "shotId": 83
      },
      {
        "id": 16227,
        "t": 8.19524385966651,
        "x": 280.323794327026,
        "y": 0.04965261000250346,
        "z": -256.7416668078187,
        "shotId": 83
      },
      {
        "id": 16228,
        "t": 8.21524385966651,
        "x": 280.3901236436612,
        "y": 0.04668517042757175,
        "z": -256.8081323440313,
        "shotId": 83
      },
      {
        "id": 16229,
        "t": 8.235243859666511,
        "x": 280.4564360102408,
        "y": 0.03942090521588733,
        "z": -256.8745808953783,
        "shotId": 83
      },
      {
        "id": 16230,
        "t": 8.25524385966651,
        "x": 280.522730603478,
        "y": 0.02786079842827027,
        "z": -256.941011636882,
        "shotId": 83
      },
      {
        "id": 16231,
        "t": 8.27524385966651,
        "x": 280.5890065279252,
        "y": 0.01200581235540408,
        "z": -257.007423671256,
        "shotId": 83
      },
      {
        "id": 16232,
        "t": 8.287160941581812,
        "x": 280.6284856051653,
        "y": -2.428279799460143e-16,
        "z": -257.0469838261175,
        "shotId": 83
      },
      {
        "id": 16233,
        "t": 8.287160941581812,
        "x": 280.6284856051653,
        "y": -2.428279799460143e-16,
        "z": -257.0469838261175,
        "shotId": 83
      },
      {
        "id": 16234,
        "t": 8.307160941581811,
        "x": 280.6901993720614,
        "y": 0.009003951463363075,
        "z": -257.108824333702,
        "shotId": 83
      },
      {
        "id": 16235,
        "t": 8.32716094158181,
        "x": 280.7519002538469,
        "y": 0.01370999909313243,
        "z": -257.1706519297139,
        "shotId": 83
      },
      {
        "id": 16236,
        "t": 8.347160941581812,
        "x": 280.8135876408379,
        "y": 0.01411917472151845,
        "z": -257.2324660032172,
        "shotId": 83
      },
      {
        "id": 16237,
        "t": 8.367160941581812,
        "x": 280.8752608713984,
        "y": 0.01023245595957016,
        "z": -257.2942658912174,
        "shotId": 83
      },
      {
        "id": 16238,
        "t": 8.387160941581811,
        "x": 280.9369192235751,
        "y": 0.002050778218339532,
        "z": -257.356050870278,
        "shotId": 83
      },
      {
        "id": 16239,
        "t": 8.390448569720634,
        "x": 280.9470521346892,
        "y": -2.428279799460143e-16,
        "z": -257.366204591208,
        "shotId": 83
      },
      {
        "id": 16240,
        "t": 8.390448569720634,
        "x": 280.9470521346892,
        "y": -2.428279799460143e-16,
        "z": -257.366204591208,
        "shotId": 83
      },
      {
        "id": 16241,
        "t": 8.417976552238203,
        "x": 281.0265674641531,
        "y": -2.428279799460143e-16,
        "z": -257.445883220177,
        "shotId": 83
      },
      {
        "id": 16242,
        "t": 8.445504534755774,
        "x": 281.1049863109701,
        "y": -2.428279799460143e-16,
        "z": -257.5244631146684,
        "shotId": 83
      },
      {
        "id": 16243,
        "t": 8.473032517273342,
        "x": 281.1826008619224,
        "y": -2.428279799460143e-16,
        "z": -257.6022370615239,
        "shotId": 83
      },
      {
        "id": 16244,
        "t": 8.500560499790911,
        "x": 281.2594111170098,
        "y": -2.428279799460143e-16,
        "z": -257.6792050607436,
        "shotId": 83
      },
      {
        "id": 16245,
        "t": 8.528088482308481,
        "x": 281.3354170762325,
        "y": -2.428279799460143e-16,
        "z": -257.7553671123275,
        "shotId": 83
      },
      {
        "id": 16246,
        "t": 8.55561646482605,
        "x": 281.4106187395903,
        "y": -2.428279799460143e-16,
        "z": -257.8307232162756,
        "shotId": 83
      },
      {
        "id": 16247,
        "t": 8.58314444734362,
        "x": 281.4850161070835,
        "y": -2.428279799460143e-16,
        "z": -257.9052733725879,
        "shotId": 83
      },
      {
        "id": 16248,
        "t": 8.61067242986119,
        "x": 281.5586091787118,
        "y": -2.428279799460143e-16,
        "z": -257.9790175812644,
        "shotId": 83
      },
      {
        "id": 16249,
        "t": 8.638200412378758,
        "x": 281.6313979544753,
        "y": -2.428279799460143e-16,
        "z": -258.0519558423051,
        "shotId": 83
      },
      {
        "id": 16250,
        "t": 8.665728394896329,
        "x": 281.703382434374,
        "y": -2.428279799460143e-16,
        "z": -258.12408815571,
        "shotId": 83
      },
      {
        "id": 16251,
        "t": 8.693256377413897,
        "x": 281.774562618408,
        "y": -2.428279799460143e-16,
        "z": -258.195414521479,
        "shotId": 83
      },
      {
        "id": 16252,
        "t": 8.720784359931466,
        "x": 281.8449385065771,
        "y": -2.428279799460143e-16,
        "z": -258.2659349396122,
        "shotId": 83
      },
      {
        "id": 16253,
        "t": 8.748312342449037,
        "x": 281.9145100988815,
        "y": -2.428279799460143e-16,
        "z": -258.3356494101097,
        "shotId": 83
      },
      {
        "id": 16254,
        "t": 8.775840324966605,
        "x": 281.9832773953211,
        "y": -2.428279799460143e-16,
        "z": -258.4045579329713,
        "shotId": 83
      },
      {
        "id": 16255,
        "t": 8.803368307484176,
        "x": 282.0512403958958,
        "y": -2.428279799460143e-16,
        "z": -258.4726605081971,
        "shotId": 83
      },
      {
        "id": 16256,
        "t": 8.830896290001744,
        "x": 282.1183991006058,
        "y": -2.428279799460143e-16,
        "z": -258.5399571357871,
        "shotId": 83
      },
      {
        "id": 16257,
        "t": 8.858424272519313,
        "x": 282.184753509451,
        "y": -2.428279799460143e-16,
        "z": -258.6064478157414,
        "shotId": 83
      },
      {
        "id": 16258,
        "t": 8.885952255036884,
        "x": 282.2503036224314,
        "y": -2.428279799460143e-16,
        "z": -258.6721325480598,
        "shotId": 83
      },
      {
        "id": 16259,
        "t": 8.913480237554452,
        "x": 282.315049439547,
        "y": -2.428279799460143e-16,
        "z": -258.7370113327423,
        "shotId": 83
      },
      {
        "id": 16260,
        "t": 8.941008220072021,
        "x": 282.3789909607979,
        "y": -2.428279799460143e-16,
        "z": -258.8010841697891,
        "shotId": 83
      },
      {
        "id": 16261,
        "t": 8.968536202589592,
        "x": 282.4421281861839,
        "y": -2.428279799460143e-16,
        "z": -258.8643510592001,
        "shotId": 83
      },
      {
        "id": 16262,
        "t": 8.99606418510716,
        "x": 282.5044611157052,
        "y": -2.428279799460143e-16,
        "z": -258.9268120009752,
        "shotId": 83
      },
      {
        "id": 16263,
        "t": 9.02359216762473,
        "x": 282.5659897493616,
        "y": -2.428279799460143e-16,
        "z": -258.9884669951146,
        "shotId": 83
      },
      {
        "id": 16264,
        "t": 9.0511201501423,
        "x": 282.6267140871532,
        "y": -2.428279799460143e-16,
        "z": -259.0493160416182,
        "shotId": 83
      },
      {
        "id": 16265,
        "t": 9.078648132659868,
        "x": 282.6866341290801,
        "y": -2.428279799460143e-16,
        "z": -259.1093591404859,
        "shotId": 83
      },
      {
        "id": 16266,
        "t": 9.106176115177439,
        "x": 282.7457498751422,
        "y": -2.428279799460143e-16,
        "z": -259.1685962917178,
        "shotId": 83
      },
      {
        "id": 16267,
        "t": 9.133704097695007,
        "x": 282.8040613253395,
        "y": -2.428279799460143e-16,
        "z": -259.2270274953139,
        "shotId": 83
      },
      {
        "id": 16268,
        "t": 9.161232080212576,
        "x": 282.861568479672,
        "y": -2.428279799460143e-16,
        "z": -259.2846527512742,
        "shotId": 83
      },
      {
        "id": 16269,
        "t": 9.188760062730147,
        "x": 282.9182713381397,
        "y": -2.428279799460143e-16,
        "z": -259.3414720595987,
        "shotId": 83
      },
      {
        "id": 16270,
        "t": 9.216288045247715,
        "x": 282.9741699007426,
        "y": -2.428279799460143e-16,
        "z": -259.3974854202875,
        "shotId": 83
      },
      {
        "id": 16271,
        "t": 9.243816027765284,
        "x": 283.0292641674807,
        "y": -2.428279799460143e-16,
        "z": -259.4526928333403,
        "shotId": 83
      },
      {
        "id": 16272,
        "t": 9.271344010282855,
        "x": 283.0835541383541,
        "y": -2.428279799460143e-16,
        "z": -259.5070942987574,
        "shotId": 83
      },
      {
        "id": 16273,
        "t": 9.298871992800423,
        "x": 283.1370398133627,
        "y": -2.428279799460143e-16,
        "z": -259.5606898165387,
        "shotId": 83
      },
      {
        "id": 16274,
        "t": 9.326399975317994,
        "x": 283.1897211925064,
        "y": -2.428279799460143e-16,
        "z": -259.6134793866842,
        "shotId": 83
      },
      {
        "id": 16275,
        "t": 9.353927957835563,
        "x": 283.2415982757854,
        "y": -2.428279799460143e-16,
        "z": -259.6654630091938,
        "shotId": 83
      },
      {
        "id": 16276,
        "t": 9.381455940353131,
        "x": 283.2926710631996,
        "y": -2.428279799460143e-16,
        "z": -259.7166406840677,
        "shotId": 83
      },
      {
        "id": 16277,
        "t": 9.408983922870702,
        "x": 283.342939554749,
        "y": -2.428279799460143e-16,
        "z": -259.7670124113056,
        "shotId": 83
      },
      {
        "id": 16278,
        "t": 9.43651190538827,
        "x": 283.3924037504335,
        "y": -2.428279799460143e-16,
        "z": -259.8165781909079,
        "shotId": 83
      },
      {
        "id": 16279,
        "t": 9.464039887905841,
        "x": 283.4410636502533,
        "y": -2.428279799460143e-16,
        "z": -259.8653380228743,
        "shotId": 83
      },
      {
        "id": 16280,
        "t": 9.49156787042341,
        "x": 283.4889192542084,
        "y": -2.428279799460143e-16,
        "z": -259.9132919072049,
        "shotId": 83
      },
      {
        "id": 16281,
        "t": 9.519095852940978,
        "x": 283.5359705622986,
        "y": -2.428279799460143e-16,
        "z": -259.9604398438997,
        "shotId": 83
      },
      {
        "id": 16282,
        "t": 9.546623835458549,
        "x": 283.5822175745241,
        "y": -2.428279799460143e-16,
        "z": -260.0067818329587,
        "shotId": 83
      },
      {
        "id": 16283,
        "t": 9.574151817976118,
        "x": 283.6276602908847,
        "y": -2.428279799460143e-16,
        "z": -260.0523178743819,
        "shotId": 83
      },
      {
        "id": 16284,
        "t": 9.601679800493686,
        "x": 283.6722987113806,
        "y": -2.428279799460143e-16,
        "z": -260.0970479681692,
        "shotId": 83
      },
      {
        "id": 16285,
        "t": 9.629207783011257,
        "x": 283.7161328360116,
        "y": -2.428279799460143e-16,
        "z": -260.1409721143208,
        "shotId": 83
      },
      {
        "id": 16286,
        "t": 9.656735765528826,
        "x": 283.7591626647779,
        "y": -2.428279799460143e-16,
        "z": -260.1840903128366,
        "shotId": 83
      },
      {
        "id": 16287,
        "t": 9.684263748046394,
        "x": 283.8013881976794,
        "y": -2.428279799460143e-16,
        "z": -260.2264025637166,
        "shotId": 83
      },
      {
        "id": 16288,
        "t": 9.711791730563965,
        "x": 283.8428094347161,
        "y": -2.428279799460143e-16,
        "z": -260.2679088669607,
        "shotId": 83
      },
      {
        "id": 16289,
        "t": 9.739319713081533,
        "x": 283.883426375888,
        "y": -2.428279799460143e-16,
        "z": -260.308609222569,
        "shotId": 83
      },
      {
        "id": 16290,
        "t": 9.766847695599104,
        "x": 283.9232390211951,
        "y": -2.428279799460143e-16,
        "z": -260.3485036305416,
        "shotId": 83
      },
      {
        "id": 16291,
        "t": 9.794375678116673,
        "x": 283.9622473706374,
        "y": -2.428279799460143e-16,
        "z": -260.3875920908783,
        "shotId": 83
      },
      {
        "id": 16292,
        "t": 9.821903660634241,
        "x": 284.000451424215,
        "y": -2.428279799460143e-16,
        "z": -260.4258746035792,
        "shotId": 83
      },
      {
        "id": 16293,
        "t": 9.849431643151812,
        "x": 284.0378511819277,
        "y": -2.428279799460143e-16,
        "z": -260.4633511686443,
        "shotId": 83
      },
      {
        "id": 16294,
        "t": 9.87695962566938,
        "x": 284.0744466437757,
        "y": -2.428279799460143e-16,
        "z": -260.5000217860735,
        "shotId": 83
      },
      {
        "id": 16295,
        "t": 9.90448760818695,
        "x": 284.1102378097589,
        "y": -2.428279799460143e-16,
        "z": -260.5358864558671,
        "shotId": 83
      },
      {
        "id": 16296,
        "t": 9.93201559070452,
        "x": 284.1452246798772,
        "y": -2.428279799460143e-16,
        "z": -260.5709451780247,
        "shotId": 83
      },
      {
        "id": 16297,
        "t": 9.959543573222088,
        "x": 284.1794072541309,
        "y": -2.428279799460143e-16,
        "z": -260.6051979525466,
        "shotId": 83
      },
      {
        "id": 16298,
        "t": 9.987071555739657,
        "x": 284.2127855325197,
        "y": -2.428279799460143e-16,
        "z": -260.6386447794326,
        "shotId": 83
      },
      {
        "id": 16299,
        "t": 10.01459953825723,
        "x": 284.2453595150436,
        "y": -2.428279799460143e-16,
        "z": -260.6712856586829,
        "shotId": 83
      },
      {
        "id": 16300,
        "t": 10.0421275207748,
        "x": 284.2771292017028,
        "y": -2.428279799460143e-16,
        "z": -260.7031205902973,
        "shotId": 83
      },
      {
        "id": 16301,
        "t": 10.06965550329237,
        "x": 284.3080945924973,
        "y": -2.428279799460143e-16,
        "z": -260.7341495742759,
        "shotId": 83
      },
      {
        "id": 16302,
        "t": 10.09718348580994,
        "x": 284.3382556874269,
        "y": -2.428279799460143e-16,
        "z": -260.7643726106188,
        "shotId": 83
      },
      {
        "id": 16303,
        "t": 10.1247114683275,
        "x": 284.3676124864917,
        "y": -2.428279799460143e-16,
        "z": -260.7937896993258,
        "shotId": 83
      },
      {
        "id": 16304,
        "t": 10.15223945084507,
        "x": 284.3961649896918,
        "y": -2.428279799460143e-16,
        "z": -260.822400840397,
        "shotId": 83
      },
      {
        "id": 16305,
        "t": 10.17976743336264,
        "x": 284.423913197027,
        "y": -2.428279799460143e-16,
        "z": -260.8502060338324,
        "shotId": 83
      },
      {
        "id": 16306,
        "t": 10.20729541588021,
        "x": 284.4508571084975,
        "y": -2.428279799460143e-16,
        "z": -260.877205279632,
        "shotId": 83
      },
      {
        "id": 16307,
        "t": 10.23482339839778,
        "x": 284.4769967241032,
        "y": -2.428279799460143e-16,
        "z": -260.9033985777957,
        "shotId": 83
      },
      {
        "id": 16308,
        "t": 10.26235138091535,
        "x": 284.5023320438441,
        "y": -2.428279799460143e-16,
        "z": -260.9287859283237,
        "shotId": 83
      },
      {
        "id": 16309,
        "t": 10.28987936343292,
        "x": 284.5268630677202,
        "y": -2.428279799460143e-16,
        "z": -260.9533673312159,
        "shotId": 83
      },
      {
        "id": 16310,
        "t": 10.31740734595049,
        "x": 284.5505897957315,
        "y": -2.428279799460143e-16,
        "z": -260.9771427864723,
        "shotId": 83
      },
      {
        "id": 16311,
        "t": 10.34493532846806,
        "x": 284.573512227878,
        "y": -2.428279799460143e-16,
        "z": -261.0001122940928,
        "shotId": 83
      },
      {
        "id": 16312,
        "t": 10.37246331098563,
        "x": 284.5956303641598,
        "y": -2.428279799460143e-16,
        "z": -261.0222758540775,
        "shotId": 83
      },
      {
        "id": 16313,
        "t": 10.3999912935032,
        "x": 284.6169442045767,
        "y": -2.428279799460143e-16,
        "z": -261.0436334664265,
        "shotId": 83
      },
      {
        "id": 16314,
        "t": 10.42751927602077,
        "x": 284.6374537491289,
        "y": -2.428279799460143e-16,
        "z": -261.0641851311395,
        "shotId": 83
      },
      {
        "id": 16315,
        "t": 10.45504725853834,
        "x": 284.6571589978162,
        "y": -2.428279799460143e-16,
        "z": -261.0839308482169,
        "shotId": 83
      },
      {
        "id": 16316,
        "t": 10.48257524105591,
        "x": 284.6760599506388,
        "y": -2.428279799460143e-16,
        "z": -261.1028706176584,
        "shotId": 83
      },
      {
        "id": 16317,
        "t": 10.51010322357348,
        "x": 284.6941566075966,
        "y": -2.428279799460143e-16,
        "z": -261.1210044394641,
        "shotId": 83
      },
      {
        "id": 16318,
        "t": 10.53763120609105,
        "x": 284.7114489686896,
        "y": -2.428279799460143e-16,
        "z": -261.138332313634,
        "shotId": 83
      },
      {
        "id": 16319,
        "t": 10.56515918860861,
        "x": 284.7279370339177,
        "y": -2.428279799460143e-16,
        "z": -261.154854240168,
        "shotId": 83
      },
      {
        "id": 16320,
        "t": 10.59268717112618,
        "x": 284.7436208032812,
        "y": -2.428279799460143e-16,
        "z": -261.1705702190663,
        "shotId": 83
      },
      {
        "id": 16321,
        "t": 10.62021515364375,
        "x": 284.7585002767798,
        "y": -2.428279799460143e-16,
        "z": -261.1854802503287,
        "shotId": 83
      },
      {
        "id": 16322,
        "t": 10.64774313616132,
        "x": 284.7725754544136,
        "y": -2.428279799460143e-16,
        "z": -261.1995843339554,
        "shotId": 83
      },
      {
        "id": 16323,
        "t": 10.67527111867889,
        "x": 284.7858463361827,
        "y": -2.428279799460143e-16,
        "z": -261.2128824699462,
        "shotId": 83
      },
      {
        "id": 16324,
        "t": 10.70279910119646,
        "x": 284.7983129220869,
        "y": -2.428279799460143e-16,
        "z": -261.2253746583012,
        "shotId": 83
      },
      {
        "id": 16325,
        "t": 10.73032708371403,
        "x": 284.8099752121264,
        "y": -2.428279799460143e-16,
        "z": -261.2370608990205,
        "shotId": 83
      },
      {
        "id": 16326,
        "t": 10.7578550662316,
        "x": 284.820833206301,
        "y": -2.428279799460143e-16,
        "z": -261.2479411921039,
        "shotId": 83
      },
      {
        "id": 16327,
        "t": 10.78538304874917,
        "x": 284.8308869046109,
        "y": -2.428279799460143e-16,
        "z": -261.2580155375515,
        "shotId": 83
      },
      {
        "id": 16328,
        "t": 10.81291103126674,
        "x": 284.840136307056,
        "y": -2.428279799460143e-16,
        "z": -261.2672839353633,
        "shotId": 83
      },
      {
        "id": 16329,
        "t": 10.84043901378431,
        "x": 284.8485814136363,
        "y": -2.428279799460143e-16,
        "z": -261.2757463855393,
        "shotId": 83
      },
      {
        "id": 16330,
        "t": 10.86796699630188,
        "x": 284.8562222243518,
        "y": -2.428279799460143e-16,
        "z": -261.2834028880795,
        "shotId": 83
      },
      {
        "id": 16331,
        "t": 10.89549497881945,
        "x": 284.8630587392025,
        "y": -2.428279799460143e-16,
        "z": -261.2902534429838,
        "shotId": 83
      },
      {
        "id": 16332,
        "t": 10.92302296133702,
        "x": 284.8690909581885,
        "y": -2.428279799460143e-16,
        "z": -261.2962980502525,
        "shotId": 83
      },
      {
        "id": 16333,
        "t": 10.95055094385459,
        "x": 284.8743188813096,
        "y": -2.428279799460143e-16,
        "z": -261.3015367098852,
        "shotId": 83
      },
      {
        "id": 16334,
        "t": 10.97807892637216,
        "x": 284.8787425085659,
        "y": -2.428279799460143e-16,
        "z": -261.3059694218821,
        "shotId": 83
      },
      {
        "id": 16335,
        "t": 11.00560690888972,
        "x": 284.8823618399575,
        "y": -2.428279799460143e-16,
        "z": -261.3095961862433,
        "shotId": 83
      },
      {
        "id": 16336,
        "t": 11.03313489140729,
        "x": 284.8851768754843,
        "y": -2.428279799460143e-16,
        "z": -261.3124170029686,
        "shotId": 83
      },
      {
        "id": 16337,
        "t": 11.06066287392486,
        "x": 284.8871876151463,
        "y": -2.428279799460143e-16,
        "z": -261.3144318720581,
        "shotId": 83
      },
      {
        "id": 16338,
        "t": 11.08819085644243,
        "x": 284.8883940589434,
        "y": -2.428279799460143e-16,
        "z": -261.3156407935118,
        "shotId": 83
      },
      {
        "id": 16339,
        "t": 11.11571883896,
        "x": 284.8887962068758,
        "y": -2.428279799460143e-16,
        "z": -261.3160437673297,
        "shotId": 83
      }
    ]
  },
  {
    "id": 84,
    "number": 3,
    "ballMph": 180,
    "launchAngle": 20,
    "verticalRpm": 2222,
    "horizontalRpm": 0,
    "carryYards": 0,
    "totalYards": 0,
    "landingAngle": 0,
    "deviationAngle": -7.5,
    "smashFactor": null,
    "ignore": false,
    "manuallyEntered": false,
    "rdPeakHeightYards": 66.18697814431809,
    "rdCarryYards": 309.1150362302757,
    "rdRollYards": 317.5632030519345,
    "rdDevCarryYards": -44.70844071296911,
    "rdDevRollYards": -45.67182018711492,
    "rdLandingDeg": -50.91379240637124,
    "carryIdx": 88,
    "rollIdx": 286,
    "clubConfigId": 54,
    "average": true,
    "shotGraphs": [
      {
        "id": 16340,
        "t": 0,
        "x": 0,
        "y": 0,
        "z": 0,
        "shotId": 84
      },
      {
        "id": 16341,
        "t": 0.1,
        "x": 8.125648035750434,
        "y": 2.986981443403024,
        "z": -1.062414049815539,
        "shotId": 84
      },
      {
        "id": 16342,
        "t": 0.2,
        "x": 15.94676913300311,
        "y": 5.925158173982197,
        "z": -2.092355387893154,
        "shotId": 84
      },
      {
        "id": 16343,
        "t": 0.3,
        "x": 23.482360846168,
        "y": 8.81024683780937,
        "z": -3.091808237791806,
        "shotId": 84
      },
      {
        "id": 16344,
        "t": 0.4,
        "x": 30.74965054736426,
        "y": 11.63875259110686,
        "z": -4.062569418057392,
        "shotId": 84
      },
      {
        "id": 16345,
        "t": 0.5,
        "x": 37.76437697570021,
        "y": 14.40777518317394,
        "z": -5.00627661860883,
        "shotId": 84
      },
      {
        "id": 16346,
        "t": 0.6,
        "x": 44.5412999185157,
        "y": 17.11460469264289,
        "z": -5.924451528927331,
        "shotId": 84
      },
      {
        "id": 16347,
        "t": 0.7,
        "x": 51.09452094485076,
        "y": 19.75656317112071,
        "z": -6.818530980197543,
        "shotId": 84
      },
      {
        "id": 16348,
        "t": 0.7999999999999999,
        "x": 57.43716343163234,
        "y": 22.33134926034027,
        "z": -7.689847743806976,
        "shotId": 84
      },
      {
        "id": 16349,
        "t": 0.8999999999999999,
        "x": 63.58141143628999,
        "y": 24.83702588002508,
        "z": -8.539636795145658,
        "shotId": 84
      },
      {
        "id": 16350,
        "t": 0.9999999999999999,
        "x": 69.53869956171424,
        "y": 27.27190794567119,
        "z": -9.369053821992184,
        "shotId": 84
      },
      {
        "id": 16351,
        "t": 1.1,
        "x": 75.31971999199877,
        "y": 29.63457406315278,
        "z": -10.17917803000234,
        "shotId": 84
      },
      {
        "id": 16352,
        "t": 1.2,
        "x": 80.93433893425212,
        "y": 31.92389147942259,
        "z": -10.97100454173368,
        "shotId": 84
      },
      {
        "id": 16353,
        "t": 1.3,
        "x": 86.39167520980666,
        "y": 34.13895706308377,
        "z": -11.74545171079041,
        "shotId": 84
      },
      {
        "id": 16354,
        "t": 1.4,
        "x": 91.70023964511138,
        "y": 36.27904108957734,
        "z": -12.5033755512794,
        "shotId": 84
      },
      {
        "id": 16355,
        "t": 1.5,
        "x": 96.86791223161552,
        "y": 38.3436125555842,
        "z": -13.24556850035593,
        "shotId": 84
      },
      {
        "id": 16356,
        "t": 1.6,
        "x": 101.9017199671189,
        "y": 40.33245039081826,
        "z": -13.97273763496743,
        "shotId": 84
      },
      {
        "id": 16357,
        "t": 1.7,
        "x": 106.8077694444835,
        "y": 42.24567584582063,
        "z": -14.68549719861949,
        "shotId": 84
      },
      {
        "id": 16358,
        "t": 1.8,
        "x": 111.5913730948767,
        "y": 44.08368042854207,
        "z": -15.3843796891682,
        "shotId": 84
      },
      {
        "id": 16359,
        "t": 1.900000000000001,
        "x": 116.2572803128638,
        "y": 45.84699553432426,
        "z": -16.0698571899779,
        "shotId": 84
      },
      {
        "id": 16360,
        "t": 2,
        "x": 120.8099032479608,
        "y": 47.53616897969204,
        "z": -16.74236294161429,
        "shotId": 84
      },
      {
        "id": 16361,
        "t": 2.100000000000001,
        "x": 125.2534233108837,
        "y": 49.15171195638971,
        "z": -17.40230184038311,
        "shotId": 84
      },
      {
        "id": 16362,
        "t": 2.200000000000001,
        "x": 129.5918625561979,
        "y": 50.69406849565748,
        "z": -18.05005784816848,
        "shotId": 84
      },
      {
        "id": 16363,
        "t": 2.300000000000001,
        "x": 133.829135676371,
        "y": 52.16359265618448,
        "z": -18.68599948272088,
        "shotId": 84
      },
      {
        "id": 16364,
        "t": 2.400000000000001,
        "x": 137.9690797171233,
        "y": 53.56054515659895,
        "z": -19.31048362803367,
        "shotId": 84
      },
      {
        "id": 16365,
        "t": 2.500000000000001,
        "x": 142.0154655116585,
        "y": 54.88509144088821,
        "z": -19.923857066367,
        "shotId": 84
      },
      {
        "id": 16366,
        "t": 2.600000000000001,
        "x": 145.972013648013,
        "y": 56.1373034615602,
        "z": -20.52645898364565,
        "shotId": 84
      },
      {
        "id": 16367,
        "t": 2.700000000000001,
        "x": 149.842386246896,
        "y": 57.31717360791914,
        "z": -21.11862074625617,
        "shotId": 84
      },
      {
        "id": 16368,
        "t": 2.800000000000001,
        "x": 153.6301931398577,
        "y": 58.42461498521283,
        "z": -21.70066721224464,
        "shotId": 84
      },
      {
        "id": 16369,
        "t": 2.900000000000001,
        "x": 157.3390134604172,
        "y": 59.45943449391981,
        "z": -22.27291908412606,
        "shotId": 84
      },
      {
        "id": 16370,
        "t": 3.000000000000001,
        "x": 160.9723854822015,
        "y": 60.42133928057887,
        "z": -22.83569240379549,
        "shotId": 84
      },
      {
        "id": 16371,
        "t": 3.100000000000001,
        "x": 164.5337447283143,
        "y": 61.31002240747652,
        "z": -23.38929364587649,
        "shotId": 84
      },
      {
        "id": 16372,
        "t": 3.200000000000002,
        "x": 168.026370139803,
        "y": 62.12516978023905,
        "z": -23.93401249432079,
        "shotId": 84
      },
      {
        "id": 16373,
        "t": 3.300000000000002,
        "x": 171.4533823098941,
        "y": 62.86646484101463,
        "z": -24.47012170104222,
        "shotId": 84
      },
      {
        "id": 16374,
        "t": 3.400000000000002,
        "x": 174.8177553259661,
        "y": 63.53364200915273,
        "z": -24.9978806711889,
        "shotId": 84
      },
      {
        "id": 16375,
        "t": 3.500000000000002,
        "x": 178.122295024096,
        "y": 64.12646862155061,
        "z": -25.51753137587648,
        "shotId": 84
      },
      {
        "id": 16376,
        "t": 3.600000000000002,
        "x": 181.3696775680841,
        "y": 64.64476725531253,
        "z": -26.02930456761498,
        "shotId": 84
      },
      {
        "id": 16377,
        "t": 3.700000000000002,
        "x": 184.5624419515777,
        "y": 65.08842254192867,
        "z": -26.53341844170228,
        "shotId": 84
      },
      {
        "id": 16378,
        "t": 3.800000000000002,
        "x": 187.7029910524878,
        "y": 65.45737842894569,
        "z": -27.03007828608612,
        "shotId": 84
      },
      {
        "id": 16379,
        "t": 3.900000000000002,
        "x": 190.7936209485073,
        "y": 65.75162905276592,
        "z": -27.5194803569067,
        "shotId": 84
      },
      {
        "id": 16380,
        "t": 4.000000000000002,
        "x": 193.8365181054351,
        "y": 65.97122673239532,
        "z": -28.001811097452,
        "shotId": 84
      },
      {
        "id": 16381,
        "t": 4.100000000000001,
        "x": 196.8337566182657,
        "y": 66.11628977245466,
        "z": -28.47724626267196,
        "shotId": 84
      },
      {
        "id": 16382,
        "t": 4.200000000000001,
        "x": 199.7873172581682,
        "y": 66.18697814431809,
        "z": -28.94595307739683,
        "shotId": 84
      },
      {
        "id": 16383,
        "t": 4.300000000000001,
        "x": 202.6990933660043,
        "y": 66.18348778438177,
        "z": -29.40809068568804,
        "shotId": 84
      },
      {
        "id": 16384,
        "t": 4.4,
        "x": 205.5708918896208,
        "y": 66.1060597658261,
        "z": -29.86380986690884,
        "shotId": 84
      },
      {
        "id": 16385,
        "t": 4.5,
        "x": 208.4044338619597,
        "y": 65.95499796444113,
        "z": -30.31325252639291,
        "shotId": 84
      },
      {
        "id": 16386,
        "t": 4.6,
        "x": 211.2013644794822,
        "y": 65.73064058375121,
        "z": -30.75655286498941,
        "shotId": 84
      },
      {
        "id": 16387,
        "t": 4.699999999999999,
        "x": 213.9632577760938,
        "y": 65.43335286494276,
        "z": -31.19383779768088,
        "shotId": 84
      },
      {
        "id": 16388,
        "t": 4.799999999999999,
        "x": 216.6916197741726,
        "y": 65.06352741152965,
        "z": -31.62522713482365,
        "shotId": 84
      },
      {
        "id": 16389,
        "t": 4.899999999999999,
        "x": 219.387891768501,
        "y": 64.62158450152577,
        "z": -32.05083379696256,
        "shotId": 84
      },
      {
        "id": 16390,
        "t": 4.999999999999998,
        "x": 222.0534537032753,
        "y": 64.10797235501443,
        "z": -32.47076405995021,
        "shotId": 84
      },
      {
        "id": 16391,
        "t": 5.099999999999998,
        "x": 224.6896277211724,
        "y": 63.52316849507129,
        "z": -32.88511780559047,
        "shotId": 84
      },
      {
        "id": 16392,
        "t": 5.199999999999998,
        "x": 227.2976826370369,
        "y": 62.86768855691052,
        "z": -33.29398864588326,
        "shotId": 84
      },
      {
        "id": 16393,
        "t": 5.299999999999997,
        "x": 229.8788369583104,
        "y": 62.14207972683545,
        "z": -33.69746435059557,
        "shotId": 84
      },
      {
        "id": 16394,
        "t": 5.399999999999997,
        "x": 232.4342613721803,
        "y": 61.34691219212466,
        "z": -34.09562735818005,
        "shotId": 84
      },
      {
        "id": 16395,
        "t": 5.499999999999996,
        "x": 234.9650818560847,
        "y": 60.48277908503601,
        "z": -34.48855513409554,
        "shotId": 84
      },
      {
        "id": 16396,
        "t": 5.599999999999996,
        "x": 237.472382614574,
        "y": 59.5502963517815,
        "z": -34.87632052037558,
        "shotId": 84
      },
      {
        "id": 16397,
        "t": 5.699999999999996,
        "x": 239.9572088425254,
        "y": 58.55010254132826,
        "z": -35.25899207582923,
        "shotId": 84
      },
      {
        "id": 16398,
        "t": 5.799999999999995,
        "x": 242.4205693149534,
        "y": 57.48285851103341,
        "z": -35.63663440617307,
        "shotId": 84
      },
      {
        "id": 16399,
        "t": 5.899999999999995,
        "x": 244.8634388041823,
        "y": 56.34924704810975,
        "z": -36.0093084833558,
        "shotId": 84
      },
      {
        "id": 16400,
        "t": 5.999999999999995,
        "x": 247.2867603258685,
        "y": 55.14997240771745,
        "z": -36.37707195334145,
        "shotId": 84
      },
      {
        "id": 16401,
        "t": 6.099999999999994,
        "x": 249.691450151635,
        "y": 53.8857664955548,
        "z": -36.73997935896772,
        "shotId": 84
      },
      {
        "id": 16402,
        "t": 6.199999999999994,
        "x": 252.0784016338515,
        "y": 52.55739057682857,
        "z": -37.09808245772039,
        "shotId": 84
      },
      {
        "id": 16403,
        "t": 6.299999999999994,
        "x": 254.4484845322132,
        "y": 51.16562669426252,
        "z": -37.45143064723234,
        "shotId": 84
      },
      {
        "id": 16404,
        "t": 6.399999999999993,
        "x": 256.802546470274,
        "y": 49.71127564496191,
        "z": -37.80007126899861,
        "shotId": 84
      },
      {
        "id": 16405,
        "t": 6.499999999999993,
        "x": 259.1414144262558,
        "y": 48.19515547998516,
        "z": -38.14404988692214,
        "shotId": 84
      },
      {
        "id": 16406,
        "t": 6.599999999999993,
        "x": 261.4658960342272,
        "y": 46.61809996022013,
        "z": -38.48341054743376,
        "shotId": 84
      },
      {
        "id": 16407,
        "t": 6.699999999999992,
        "x": 263.776780705956,
        "y": 44.9809569845126,
        "z": -38.8181960204674,
        "shotId": 84
      },
      {
        "id": 16408,
        "t": 6.799999999999992,
        "x": 266.074840585398,
        "y": 43.28458700513099,
        "z": -39.14844802090663,
        "shotId": 84
      },
      {
        "id": 16409,
        "t": 6.899999999999991,
        "x": 268.3608313489711,
        "y": 41.52986144455458,
        "z": -39.47420741042123,
        "shotId": 84
      },
      {
        "id": 16410,
        "t": 6.999999999999991,
        "x": 270.6354942634405,
        "y": 39.71766381155588,
        "z": -39.79551431338055,
        "shotId": 84
      },
      {
        "id": 16411,
        "t": 7.099999999999991,
        "x": 272.8995653304054,
        "y": 37.84890409152649,
        "z": -40.11240789353933,
        "shotId": 84
      },
      {
        "id": 16412,
        "t": 7.19999999999999,
        "x": 275.1537724824954,
        "y": 35.92450988551093,
        "z": -40.4249267013733,
        "shotId": 84
      },
      {
        "id": 16413,
        "t": 7.29999999999999,
        "x": 277.3988308054314,
        "y": 33.94541487238531,
        "z": -40.73310905766429,
        "shotId": 84
      },
      {
        "id": 16414,
        "t": 7.39999999999999,
        "x": 279.6354427426354,
        "y": 31.91255733519113,
        "z": -41.03699316712176,
        "shotId": 84
      },
      {
        "id": 16415,
        "t": 7.499999999999989,
        "x": 281.8642982188436,
        "y": 29.82687874558833,
        "z": -41.33661721599319,
        "shotId": 84
      },
      {
        "id": 16416,
        "t": 7.599999999999989,
        "x": 284.0860746956578,
        "y": 27.68932241094635,
        "z": -41.6320194547071,
        "shotId": 84
      },
      {
        "id": 16417,
        "t": 7.699999999999989,
        "x": 286.3014390162065,
        "y": 25.5008380677503,
        "z": -41.92323799922094,
        "shotId": 84
      },
      {
        "id": 16418,
        "t": 7.799999999999988,
        "x": 288.5110513389081,
        "y": 23.26239274948177,
        "z": -42.21031035966246,
        "shotId": 84
      },
      {
        "id": 16419,
        "t": 7.899999999999988,
        "x": 290.7155610759985,
        "y": 20.97495616376534,
        "z": -42.49327413019295,
        "shotId": 84
      },
      {
        "id": 16420,
        "t": 7.999999999999988,
        "x": 292.9156055401182,
        "y": 18.63949565122187,
        "z": -42.77216720716795,
        "shotId": 84
      },
      {
        "id": 16421,
        "t": 8.099999999999987,
        "x": 295.111809765518,
        "y": 16.25697512605744,
        "z": -43.04702781551907,
        "shotId": 84
      },
      {
        "id": 16422,
        "t": 8.199999999999987,
        "x": 297.3047863160155,
        "y": 13.82835408681121,
        "z": -43.31789452628046,
        "shotId": 84
      },
      {
        "id": 16423,
        "t": 8.299999999999986,
        "x": 299.4951341252773,
        "y": 11.35459019936464,
        "z": -43.58480600193727,
        "shotId": 84
      },
      {
        "id": 16424,
        "t": 8.399999999999986,
        "x": 301.6834347676519,
        "y": 8.836651327776146,
        "z": -43.84780005438996,
        "shotId": 84
      },
      {
        "id": 16425,
        "t": 8.499999999999986,
        "x": 303.870255089515,
        "y": 6.275504264891481,
        "z": -44.10691448102313,
        "shotId": 84
      },
      {
        "id": 16426,
        "t": 8.599999999999985,
        "x": 306.056148578094,
        "y": 3.67210813684877,
        "z": -44.36218750466475,
        "shotId": 84
      },
      {
        "id": 16427,
        "t": 8.699999999999985,
        "x": 308.2416551574194,
        "y": 1.027413680359425,
        "z": -44.61365776116618,
        "shotId": 84
      },
      {
        "id": 16428,
        "t": 8.73826421330815,
        "x": 309.0779753449858,
        "y": 0,
        "z": -44.70844071296911,
        "shotId": 84
      },
      {
        "id": 16429,
        "t": 8.73826421330815,
        "x": 309.0779753449858,
        "y": 0,
        "z": -44.70844071296911,
        "shotId": 84
      },
      {
        "id": 16430,
        "t": 8.75826421330815,
        "x": 309.1432420696513,
        "y": 0.1193398478474696,
        "z": -44.71585084090661,
        "shotId": 84
      },
      {
        "id": 16431,
        "t": 8.77826421330815,
        "x": 309.2085191980089,
        "y": 0.2343194374101714,
        "z": -44.72326215003854,
        "shotId": 84
      },
      {
        "id": 16432,
        "t": 8.798264213308151,
        "x": 309.2738049867949,
        "y": 0.3449428108348088,
        "z": -44.73067444244153,
        "shotId": 84
      },
      {
        "id": 16433,
        "t": 8.81826421330815,
        "x": 309.3390977812559,
        "y": 0.4512138767405534,
        "z": -44.73808753024139,
        "shotId": 84
      },
      {
        "id": 16434,
        "t": 8.83826421330815,
        "x": 309.4043960137207,
        "y": 0.5531364104758071,
        "z": -44.74550123545086,
        "shotId": 84
      },
      {
        "id": 16435,
        "t": 8.85826421330815,
        "x": 309.4696982020592,
        "y": 0.6507140543699679,
        "z": -44.75291538979472,
        "shotId": 84
      },
      {
        "id": 16436,
        "t": 8.878264213308151,
        "x": 309.5350029480105,
        "y": 0.7439503179848312,
        "z": -44.76032983451991,
        "shotId": 84
      },
      {
        "id": 16437,
        "t": 8.89826421330815,
        "x": 309.6003089353586,
        "y": 0.8328485783717625,
        "z": -44.76774442018841,
        "shotId": 84
      },
      {
        "id": 16438,
        "t": 8.91826421330815,
        "x": 309.6656149279287,
        "y": 0.9174120803427083,
        "z": -44.77515900644982,
        "shotId": 84
      },
      {
        "id": 16439,
        "t": 8.93826421330815,
        "x": 309.7309197673778,
        "y": 0.9976439367655963,
        "z": -44.78257346179038,
        "shotId": 84
      },
      {
        "id": 16440,
        "t": 8.958264213308151,
        "x": 309.7962223707447,
        "y": 1.073547128897834,
        "z": -44.78998766325493,
        "shotId": 84
      },
      {
        "id": 16441,
        "t": 8.978264213308151,
        "x": 309.861521727723,
        "y": 1.145124506775624,
        "z": -44.79740149613725,
        "shotId": 84
      },
      {
        "id": 16442,
        "t": 8.99826421330815,
        "x": 309.9268168976151,
        "y": 1.212378789681843,
        "z": -44.80481485363424,
        "shotId": 84
      },
      {
        "id": 16443,
        "t": 9.01826421330815,
        "x": 309.9921070059203,
        "y": 1.275312566721492,
        "z": -44.81222763645849,
        "shotId": 84
      },
      {
        "id": 16444,
        "t": 9.038264213308151,
        "x": 310.0573912405057,
        "y": 1.333928297541395,
        "z": -44.81963975240364,
        "shotId": 84
      },
      {
        "id": 16445,
        "t": 9.058264213308151,
        "x": 310.1226688473058,
        "y": 1.388228313240081,
        "z": -44.82705111585604,
        "shotId": 84
      },
      {
        "id": 16446,
        "t": 9.07826421330815,
        "x": 310.1879391254926,
        "y": 1.438214817524663,
        "z": -44.83446164724643,
        "shotId": 84
      },
      {
        "id": 16447,
        "t": 9.09826421330815,
        "x": 310.2532014220593,
        "y": 1.483889888184007,
        "z": -44.84187127243499,
        "shotId": 84
      },
      {
        "id": 16448,
        "t": 9.118264213308152,
        "x": 310.3184551257644,
        "y": 1.525255478961119,
        "z": -44.84927992202373,
        "shotId": 84
      },
      {
        "id": 16449,
        "t": 9.138264213308151,
        "x": 310.3836996603893,
        "y": 1.562313421921725,
        "z": -44.85668753059107,
        "shotId": 84
      },
      {
        "id": 16450,
        "t": 9.15826421330815,
        "x": 310.4489344772788,
        "y": 1.595065430429151,
        "z": -44.86409403584484,
        "shotId": 84
      },
      {
        "id": 16451,
        "t": 9.17826421330815,
        "x": 310.514159047157,
        "y": 1.623513102845831,
        "z": -44.87149937769316,
        "shotId": 84
      },
      {
        "id": 16452,
        "t": 9.198264213308152,
        "x": 310.5793728512388,
        "y": 1.647657927086263,
        "z": -44.87890349723514,
        "shotId": 84
      },
      {
        "id": 16453,
        "t": 9.218264213308151,
        "x": 310.6445753716978,
        "y": 1.667501286141914,
        "z": -44.88630633567888,
        "shotId": 84
      },
      {
        "id": 16454,
        "t": 9.23826421330815,
        "x": 310.7097660816016,
        "y": 1.683044464681982,
        "z": -44.89370783319849,
        "shotId": 84
      },
      {
        "id": 16455,
        "t": 9.25826421330815,
        "x": 310.7749444344648,
        "y": 1.694288656802508,
        "z": -44.90110792774818,
        "shotId": 84
      },
      {
        "id": 16456,
        "t": 9.278264213308152,
        "x": 310.840109853626,
        "y": 1.701234974949251,
        "z": -44.90850655385607,
        "shotId": 84
      },
      {
        "id": 16457,
        "t": 9.298264213308151,
        "x": 310.9052617216843,
        "y": 1.703884459978896,
        "z": -44.91590364142488,
        "shotId": 84
      },
      {
        "id": 16458,
        "t": 9.31826421330815,
        "x": 310.9703993702543,
        "y": 1.702238092253832,
        "z": -44.92329911456876,
        "shotId": 84
      },
      {
        "id": 16459,
        "t": 9.33826421330815,
        "x": 311.0355220702907,
        "y": 1.696296803596191,
        "z": -44.93069289051477,
        "shotId": 84
      },
      {
        "id": 16460,
        "t": 9.358264213308152,
        "x": 311.1006290232114,
        "y": 1.686061489866928,
        "z": -44.93808487859501,
        "shotId": 84
      },
      {
        "id": 16461,
        "t": 9.378264213308151,
        "x": 311.165719352994,
        "y": 1.671533023894569,
        "z": -44.94547497934937,
        "shotId": 84
      },
      {
        "id": 16462,
        "t": 9.39826421330815,
        "x": 311.2307920993629,
        "y": 1.652712268461839,
        "z": -44.95286308375189,
        "shotId": 84
      },
      {
        "id": 16463,
        "t": 9.41826421330815,
        "x": 311.2958462121094,
        "y": 1.629600089068268,
        "z": -44.96024907256601,
        "shotId": 84
      },
      {
        "id": 16464,
        "t": 9.438264213308152,
        "x": 311.3608805465299,
        "y": 1.602197366219569,
        "z": -44.96763281582655,
        "shotId": 84
      },
      {
        "id": 16465,
        "t": 9.458264213308151,
        "x": 311.4258938599106,
        "y": 1.570505007043423,
        "z": -44.97501417244069,
        "shotId": 84
      },
      {
        "id": 16466,
        "t": 9.478264213308151,
        "x": 311.4908848089518,
        "y": 1.5345239560879,
        "z": -44.98239298989544,
        "shotId": 84
      },
      {
        "id": 16467,
        "t": 9.49826421330815,
        "x": 311.5558519480037,
        "y": 1.494255205215378,
        "z": -44.98976910405739,
        "shotId": 84
      },
      {
        "id": 16468,
        "t": 9.518264213308152,
        "x": 311.6207937279815,
        "y": 1.449699802555623,
        "z": -44.99714233904953,
        "shotId": 84
      },
      {
        "id": 16469,
        "t": 9.538264213308151,
        "x": 311.6857084958277,
        "y": 1.40085886052315,
        "z": -45.00451250719018,
        "shotId": 84
      },
      {
        "id": 16470,
        "t": 9.558264213308151,
        "x": 311.7505944944074,
        "y": 1.347733562934978,
        "z": -45.01187940898116,
        "shotId": 84
      },
      {
        "id": 16471,
        "t": 9.57826421330815,
        "x": 311.8154498627326,
        "y": 1.290325171285691,
        "z": -45.01924283313317,
        "shotId": 84
      },
      {
        "id": 16472,
        "t": 9.598264213308152,
        "x": 311.8802726364309,
        "y": 1.228635030248973,
        "z": -45.02660255661909,
        "shotId": 84
      },
      {
        "id": 16473,
        "t": 9.618264213308152,
        "x": 311.9450607483903,
        "y": 1.162664572480093,
        "z": -45.03395834474711,
        "shotId": 84
      },
      {
        "id": 16474,
        "t": 9.638264213308151,
        "x": 312.0098120295255,
        "y": 1.092415322794275,
        "z": -45.0413099512478,
        "shotId": 84
      },
      {
        "id": 16475,
        "t": 9.65826421330815,
        "x": 312.0745242096246,
        "y": 1.017888901792883,
        "z": -45.04865711837022,
        "shotId": 84
      },
      {
        "id": 16476,
        "t": 9.678264213308152,
        "x": 312.1391949182445,
        "y": 0.9390870290043909,
        "z": -45.05599957698353,
        "shotId": 84
      },
      {
        "id": 16477,
        "t": 9.698264213308152,
        "x": 312.2038216856324,
        "y": 0.8560115256010463,
        "z": -45.06333704668172,
        "shotId": 84
      },
      {
        "id": 16478,
        "t": 9.718264213308151,
        "x": 312.2684019436578,
        "y": 0.7686643167456718,
        "z": -45.07066923588929,
        "shotId": 84
      },
      {
        "id": 16479,
        "t": 9.73826421330815,
        "x": 312.332933026742,
        "y": 0.6770474336167173,
        "z": -45.07799584196692,
        "shotId": 84
      },
      {
        "id": 16480,
        "t": 9.758264213308152,
        "x": 312.3974121727806,
        "y": 0.5811630151536386,
        "z": -45.08531655131608,
        "shotId": 84
      },
      {
        "id": 16481,
        "t": 9.778264213308152,
        "x": 312.4618365240513,
        "y": 0.4810133095591786,
        "z": -45.09263103948221,
        "shotId": 84
      },
      {
        "id": 16482,
        "t": 9.798264213308151,
        "x": 312.5262031281065,
        "y": 0.3766006755901747,
        "z": -45.09993897125597,
        "shotId": 84
      },
      {
        "id": 16483,
        "t": 9.81826421330815,
        "x": 312.5905089386496,
        "y": 0.267927583664149,
        "z": -45.10724000077282,
        "shotId": 84
      },
      {
        "id": 16484,
        "t": 9.83826421330815,
        "x": 312.6547508163941,
        "y": 0.1549966168051201,
        "z": -45.11453377161045,
        "shotId": 84
      },
      {
        "id": 16485,
        "t": 9.858264213308152,
        "x": 312.7189255299062,
        "y": 0.03781047144876103,
        "z": -45.12181991688455,
        "shotId": 84
      },
      {
        "id": 16486,
        "t": 9.864491310530216,
        "x": 312.7388846924527,
        "y": 0,
        "z": -45.12408600191875,
        "shotId": 84
      },
      {
        "id": 16487,
        "t": 9.864491310530216,
        "x": 312.7388846924527,
        "y": 0,
        "z": -45.12408600191875,
        "shotId": 84
      },
      {
        "id": 16488,
        "t": 9.884491310530215,
        "x": 312.8010702784774,
        "y": 0.04184346332711666,
        "z": -45.13114630945945,
        "shotId": 84
      },
      {
        "id": 16489,
        "t": 9.904491310530215,
        "x": 312.8632478316075,
        "y": 0.079380426606873,
        "z": -45.13820570497681,
        "shotId": 84
      },
      {
        "id": 16490,
        "t": 9.924491310530216,
        "x": 312.9254168155927,
        "y": 0.1126125822913281,
        "z": -45.14526412758705,
        "shotId": 84
      },
      {
        "id": 16491,
        "t": 9.944491310530216,
        "x": 312.9875767178035,
        "y": 0.1415415059998593,
        "z": -45.15232151908825,
        "shotId": 84
      },
      {
        "id": 16492,
        "t": 9.964491310530216,
        "x": 313.049727040276,
        "y": 0.1661686608243381,
        "z": -45.15937782294353,
        "shotId": 84
      },
      {
        "id": 16493,
        "t": 9.984491310530215,
        "x": 313.1118672900104,
        "y": 0.186495402644093,
        "z": -45.16643298317965,
        "shotId": 84
      },
      {
        "id": 16494,
        "t": 10.00449131053022,
        "x": 313.1739969686254,
        "y": 0.2025229865806067,
        "z": -45.17348694321233,
        "shotId": 84
      },
      {
        "id": 16495,
        "t": 10.02449131053022,
        "x": 313.2361155615239,
        "y": 0.2142525746915713,
        "z": -45.18053964461623,
        "shotId": 84
      },
      {
        "id": 16496,
        "t": 10.04449131053022,
        "x": 313.2982225267865,
        "y": 0.2216852449538615,
        "z": -45.18759102586397,
        "shotId": 84
      },
      {
        "id": 16497,
        "t": 10.06449131053022,
        "x": 313.3603172840569,
        "y": 0.224822001516298,
        "z": -45.19464102106414,
        "shotId": 84
      },
      {
        "id": 16498,
        "t": 10.08449131053022,
        "x": 313.4223992037107,
        "y": 0.2236637861213283,
        "z": -45.20168955873167,
        "shotId": 84
      },
      {
        "id": 16499,
        "t": 10.10449131053022,
        "x": 313.4844675966036,
        "y": 0.2182114905099796,
        "z": -45.20873656062382,
        "shotId": 84
      },
      {
        "id": 16500,
        "t": 10.12449131053022,
        "x": 313.5465217046661,
        "y": 0.2084659695493901,
        "z": -45.21578194067236,
        "shotId": 84
      },
      {
        "id": 16501,
        "t": 10.14449131053022,
        "x": 313.6085606925542,
        "y": 0.1944280547693293,
        "z": -45.22282560403561,
        "shotId": 84
      },
      {
        "id": 16502,
        "t": 10.16449131053022,
        "x": 313.670583640493,
        "y": 0.1760985679719866,
        "z": -45.22986744628592,
        "shotId": 84
      },
      {
        "id": 16503,
        "t": 10.18449131053022,
        "x": 313.7325895383659,
        "y": 0.1534783345903109,
        "z": -45.2369073527386,
        "shotId": 84
      },
      {
        "id": 16504,
        "t": 10.20449131053022,
        "x": 313.794577281026,
        "y": 0.1265681965101157,
        "z": -45.24394519791964,
        "shotId": 84
      },
      {
        "id": 16505,
        "t": 10.22449131053022,
        "x": 313.8565456647446,
        "y": 0.09536902413097609,
        "z": -45.2509808451624,
        "shotId": 84
      },
      {
        "id": 16506,
        "t": 10.24449131053022,
        "x": 313.9184933846684,
        "y": 0.0598817275096615,
        "z": -45.25801414631893,
        "shotId": 84
      },
      {
        "id": 16507,
        "t": 10.26449131053022,
        "x": 313.9804190331362,
        "y": 0.02010726649743331,
        "z": -45.26504494156892,
        "shotId": 84
      },
      {
        "id": 16508,
        "t": 10.27361840524071,
        "x": 314.0086683338959,
        "y": 0,
        "z": -45.26825225638128,
        "shotId": 84
      },
      {
        "id": 16509,
        "t": 10.27361840524071,
        "x": 314.0086683338959,
        "y": 0,
        "z": -45.26825225638128,
        "shotId": 84
      },
      {
        "id": 16510,
        "t": 10.29361840524071,
        "x": 314.0656340729105,
        "y": 0.01811202339703171,
        "z": -45.2747199229651,
        "shotId": 84
      },
      {
        "id": 16511,
        "t": 10.31361840524071,
        "x": 314.1225908515433,
        "y": 0.03192731646306698,
        "z": -45.2811865722223,
        "shotId": 84
      },
      {
        "id": 16512,
        "t": 10.33361840524071,
        "x": 314.1795381927097,
        "y": 0.04144692571377345,
        "z": -45.28765214998657,
        "shotId": 84
      },
      {
        "id": 16513,
        "t": 10.35361840524071,
        "x": 314.2364755889523,
        "y": 0.04667181639071626,
        "z": -45.29411659864319,
        "shotId": 84
      },
      {
        "id": 16514,
        "t": 10.37361840524071,
        "x": 314.2934024903968,
        "y": 0.04760288403042312,
        "z": -45.30057985576163,
        "shotId": 84
      },
      {
        "id": 16515,
        "t": 10.39361840524071,
        "x": 314.3503182931709,
        "y": 0.04424096735232046,
        "z": -45.30704185278054,
        "shotId": 84
      },
      {
        "id": 16516,
        "t": 10.41361840524071,
        "x": 314.4072223286537,
        "y": 0.03658686218406548,
        "z": -45.31350251378736,
        "shotId": 84
      },
      {
        "id": 16517,
        "t": 10.43361840524071,
        "x": 314.4641138538706,
        "y": 0.02464133604987723,
        "z": -45.31996175442764,
        "shotId": 84
      },
      {
        "id": 16518,
        "t": 10.45361840524071,
        "x": 314.5209920432487,
        "y": 0.00840514299571996,
        "z": -45.32641948096909,
        "shotId": 84
      },
      {
        "id": 16519,
        "t": 10.46180811594228,
        "x": 314.5442770035688,
        "y": 0,
        "z": -45.32906316403846,
        "shotId": 84
      },
      {
        "id": 16520,
        "t": 10.46180811594228,
        "x": 314.5442770035688,
        "y": 0,
        "z": -45.32906316403846,
        "shotId": 84
      },
      {
        "id": 16521,
        "t": 10.48180811594228,
        "x": 314.5975226601303,
        "y": 0.008714671798606525,
        "z": -45.33510846705099,
        "shotId": 84
      },
      {
        "id": 16522,
        "t": 10.50180811594228,
        "x": 314.6507595844135,
        "y": 0.01313590465861114,
        "z": -45.34115277863489,
        "shotId": 84
      },
      {
        "id": 16523,
        "t": 10.52180811594228,
        "x": 314.703987256472,
        "y": 0.01326452878457613,
        "z": -45.34719603975747,
        "shotId": 84
      },
      {
        "id": 16524,
        "t": 10.54180811594228,
        "x": 314.7572051006626,
        "y": 0.00910131968796631,
        "z": -45.35323818506249,
        "shotId": 84
      },
      {
        "id": 16525,
        "t": 10.56180811594228,
        "x": 314.8104124743613,
        "y": 0.0006470132162244555,
        "z": -45.35927914158891,
        "shotId": 84
      },
      {
        "id": 16526,
        "t": 10.56282346133815,
        "x": 314.8131130993704,
        "y": 0,
        "z": -45.35958575996038,
        "shotId": 84
      },
      {
        "id": 16527,
        "t": 10.56282346133815,
        "x": 314.8131130993704,
        "y": 0,
        "z": -45.35958575996038,
        "shotId": 84
      },
      {
        "id": 16528,
        "t": 10.58492345949783,
        "x": 314.8684663907671,
        "y": 0,
        "z": -45.3658703555658,
        "shotId": 84
      },
      {
        "id": 16529,
        "t": 10.60702345765751,
        "x": 314.9231804316801,
        "y": 0,
        "z": -45.37208237317847,
        "shotId": 84
      },
      {
        "id": 16530,
        "t": 10.62912345581719,
        "x": 314.9773333029426,
        "y": 0,
        "z": -45.37823067778999,
        "shotId": 84
      },
      {
        "id": 16531,
        "t": 10.65122345397687,
        "x": 315.0309250045547,
        "y": 0,
        "z": -45.38431526940035,
        "shotId": 84
      },
      {
        "id": 16532,
        "t": 10.67332345213655,
        "x": 315.0839555365164,
        "y": 0,
        "z": -45.39033614800955,
        "shotId": 84
      },
      {
        "id": 16533,
        "t": 10.69542345029623,
        "x": 315.1364248988278,
        "y": 0,
        "z": -45.3962933136176,
        "shotId": 84
      },
      {
        "id": 16534,
        "t": 10.71752344845591,
        "x": 315.1883330914888,
        "y": 0,
        "z": -45.40218676622449,
        "shotId": 84
      },
      {
        "id": 16535,
        "t": 10.73962344661559,
        "x": 315.2396801144994,
        "y": 0,
        "z": -45.40801650583023,
        "shotId": 84
      },
      {
        "id": 16536,
        "t": 10.76172344477527,
        "x": 315.2904659678595,
        "y": 0,
        "z": -45.4137825324348,
        "shotId": 84
      },
      {
        "id": 16537,
        "t": 10.78382344293495,
        "x": 315.3406906515694,
        "y": 0,
        "z": -45.41948484603823,
        "shotId": 84
      },
      {
        "id": 16538,
        "t": 10.80592344109463,
        "x": 315.3903541656288,
        "y": 0,
        "z": -45.4251234466405,
        "shotId": 84
      },
      {
        "id": 16539,
        "t": 10.82802343925431,
        "x": 315.4394565100378,
        "y": 0,
        "z": -45.43069833424161,
        "shotId": 84
      },
      {
        "id": 16540,
        "t": 10.85012343741399,
        "x": 315.4879976847965,
        "y": 0,
        "z": -45.43620950884157,
        "shotId": 84
      },
      {
        "id": 16541,
        "t": 10.87222343557367,
        "x": 315.5359776899047,
        "y": 0,
        "z": -45.44165697044037,
        "shotId": 84
      },
      {
        "id": 16542,
        "t": 10.89432343373335,
        "x": 315.5833965253626,
        "y": 0,
        "z": -45.44704071903802,
        "shotId": 84
      },
      {
        "id": 16543,
        "t": 10.91642343189303,
        "x": 315.63025419117,
        "y": 0,
        "z": -45.45236075463451,
        "shotId": 84
      },
      {
        "id": 16544,
        "t": 10.93852343005271,
        "x": 315.6765506873271,
        "y": 0,
        "z": -45.45761707722985,
        "shotId": 84
      },
      {
        "id": 16545,
        "t": 10.96062342821239,
        "x": 315.7222860138338,
        "y": 0,
        "z": -45.46280968682403,
        "shotId": 84
      },
      {
        "id": 16546,
        "t": 10.98272342637207,
        "x": 315.7674601706901,
        "y": 0,
        "z": -45.46793858341706,
        "shotId": 84
      },
      {
        "id": 16547,
        "t": 11.00482342453175,
        "x": 315.812073157896,
        "y": 0,
        "z": -45.47300376700893,
        "shotId": 84
      },
      {
        "id": 16548,
        "t": 11.02692342269143,
        "x": 315.8561249754515,
        "y": 0,
        "z": -45.47800523759963,
        "shotId": 84
      },
      {
        "id": 16549,
        "t": 11.04902342085111,
        "x": 315.8996156233567,
        "y": 0,
        "z": -45.48294299518919,
        "shotId": 84
      },
      {
        "id": 16550,
        "t": 11.07112341901079,
        "x": 315.9425451016114,
        "y": 0,
        "z": -45.4878170397776,
        "shotId": 84
      },
      {
        "id": 16551,
        "t": 11.09322341717047,
        "x": 315.9849134102158,
        "y": 0,
        "z": -45.49262737136485,
        "shotId": 84
      },
      {
        "id": 16552,
        "t": 11.11532341533015,
        "x": 316.0267205491698,
        "y": 0,
        "z": -45.49737398995093,
        "shotId": 84
      },
      {
        "id": 16553,
        "t": 11.13742341348983,
        "x": 316.0679665184734,
        "y": 0,
        "z": -45.50205689553587,
        "shotId": 84
      },
      {
        "id": 16554,
        "t": 11.15952341164951,
        "x": 316.1086513181266,
        "y": 0,
        "z": -45.50667608811965,
        "shotId": 84
      },
      {
        "id": 16555,
        "t": 11.18162340980919,
        "x": 316.1487749481294,
        "y": 0,
        "z": -45.51123156770227,
        "shotId": 84
      },
      {
        "id": 16556,
        "t": 11.20372340796887,
        "x": 316.1883374084817,
        "y": 0,
        "z": -45.51572333428374,
        "shotId": 84
      },
      {
        "id": 16557,
        "t": 11.22582340612855,
        "x": 316.2273386991838,
        "y": 0,
        "z": -45.52015138786406,
        "shotId": 84
      },
      {
        "id": 16558,
        "t": 11.24792340428823,
        "x": 316.2657788202354,
        "y": 0,
        "z": -45.52451572844321,
        "shotId": 84
      },
      {
        "id": 16559,
        "t": 11.27002340244791,
        "x": 316.3036577716367,
        "y": 0,
        "z": -45.52881635602122,
        "shotId": 84
      },
      {
        "id": 16560,
        "t": 11.29212340060759,
        "x": 316.3409755533876,
        "y": 0,
        "z": -45.53305327059806,
        "shotId": 84
      },
      {
        "id": 16561,
        "t": 11.31422339876727,
        "x": 316.377732165488,
        "y": 0,
        "z": -45.53722647217376,
        "shotId": 84
      },
      {
        "id": 16562,
        "t": 11.33632339692695,
        "x": 316.4139276079381,
        "y": 0,
        "z": -45.54133596074828,
        "shotId": 84
      },
      {
        "id": 16563,
        "t": 11.35842339508663,
        "x": 316.4495618807378,
        "y": 0,
        "z": -45.54538173632167,
        "shotId": 84
      },
      {
        "id": 16564,
        "t": 11.38052339324631,
        "x": 316.4846349838871,
        "y": 0,
        "z": -45.54936379889389,
        "shotId": 84
      },
      {
        "id": 16565,
        "t": 11.40262339140599,
        "x": 316.519146917386,
        "y": 0,
        "z": -45.55328214846497,
        "shotId": 84
      },
      {
        "id": 16566,
        "t": 11.42472338956567,
        "x": 316.5530976812345,
        "y": 0,
        "z": -45.55713678503488,
        "shotId": 84
      },
      {
        "id": 16567,
        "t": 11.44682338772535,
        "x": 316.5864872754326,
        "y": 0,
        "z": -45.56092770860363,
        "shotId": 84
      },
      {
        "id": 16568,
        "t": 11.46892338588503,
        "x": 316.6193156999805,
        "y": 0,
        "z": -45.56465491917124,
        "shotId": 84
      },
      {
        "id": 16569,
        "t": 11.49102338404471,
        "x": 316.6515829548778,
        "y": 0,
        "z": -45.56831841673768,
        "shotId": 84
      },
      {
        "id": 16570,
        "t": 11.51312338220439,
        "x": 316.6832890401247,
        "y": 0,
        "z": -45.57191820130297,
        "shotId": 84
      },
      {
        "id": 16571,
        "t": 11.53522338036407,
        "x": 316.7144339557213,
        "y": 0,
        "z": -45.57545427286711,
        "shotId": 84
      },
      {
        "id": 16572,
        "t": 11.55732337852375,
        "x": 316.7450177016675,
        "y": 0,
        "z": -45.57892663143009,
        "shotId": 84
      },
      {
        "id": 16573,
        "t": 11.57942337668343,
        "x": 316.7750402779633,
        "y": 0,
        "z": -45.58233527699191,
        "shotId": 84
      },
      {
        "id": 16574,
        "t": 11.60152337484312,
        "x": 316.8045016846087,
        "y": 0,
        "z": -45.58568020955258,
        "shotId": 84
      },
      {
        "id": 16575,
        "t": 11.6236233730028,
        "x": 316.8334019216038,
        "y": 0,
        "z": -45.58896142911209,
        "shotId": 84
      },
      {
        "id": 16576,
        "t": 11.64572337116248,
        "x": 316.8617409889484,
        "y": 0,
        "z": -45.59217893567045,
        "shotId": 84
      },
      {
        "id": 16577,
        "t": 11.66782336932216,
        "x": 316.8895188866426,
        "y": 0,
        "z": -45.59533272922765,
        "shotId": 84
      },
      {
        "id": 16578,
        "t": 11.68992336748184,
        "x": 316.9167356146865,
        "y": 0,
        "z": -45.5984228097837,
        "shotId": 84
      },
      {
        "id": 16579,
        "t": 11.71202336564152,
        "x": 316.94339117308,
        "y": 0,
        "z": -45.60144917733859,
        "shotId": 84
      },
      {
        "id": 16580,
        "t": 11.7341233638012,
        "x": 316.969485561823,
        "y": 0,
        "z": -45.60441183189232,
        "shotId": 84
      },
      {
        "id": 16581,
        "t": 11.75622336196088,
        "x": 316.9950187809158,
        "y": 0,
        "z": -45.6073107734449,
        "shotId": 84
      },
      {
        "id": 16582,
        "t": 11.77832336012056,
        "x": 317.019990830358,
        "y": 0,
        "z": -45.61014600199632,
        "shotId": 84
      },
      {
        "id": 16583,
        "t": 11.80042335828024,
        "x": 317.04440171015,
        "y": 0,
        "z": -45.61291751754659,
        "shotId": 84
      },
      {
        "id": 16584,
        "t": 11.82252335643992,
        "x": 317.0682514202915,
        "y": 0,
        "z": -45.6156253200957,
        "shotId": 84
      },
      {
        "id": 16585,
        "t": 11.8446233545996,
        "x": 317.0915399607827,
        "y": 0,
        "z": -45.61826940964366,
        "shotId": 84
      },
      {
        "id": 16586,
        "t": 11.86672335275928,
        "x": 317.1142673316234,
        "y": 0,
        "z": -45.62084978619046,
        "shotId": 84
      },
      {
        "id": 16587,
        "t": 11.88882335091896,
        "x": 317.1364335328138,
        "y": 0,
        "z": -45.62336644973611,
        "shotId": 84
      },
      {
        "id": 16588,
        "t": 11.91092334907864,
        "x": 317.1580385643538,
        "y": 0,
        "z": -45.6258194002806,
        "shotId": 84
      },
      {
        "id": 16589,
        "t": 11.93302334723831,
        "x": 317.1790824262433,
        "y": 0,
        "z": -45.62820863782393,
        "shotId": 84
      },
      {
        "id": 16590,
        "t": 11.955123345398,
        "x": 317.1995651184826,
        "y": 0,
        "z": -45.6305341623661,
        "shotId": 84
      },
      {
        "id": 16591,
        "t": 11.97722334355768,
        "x": 317.2194866410713,
        "y": 0,
        "z": -45.63279597390713,
        "shotId": 84
      },
      {
        "id": 16592,
        "t": 11.99932334171736,
        "x": 317.2388469940097,
        "y": 0,
        "z": -45.634994072447,
        "shotId": 84
      },
      {
        "id": 16593,
        "t": 12.02142333987704,
        "x": 317.2576461772978,
        "y": 0,
        "z": -45.63712845798572,
        "shotId": 84
      },
      {
        "id": 16594,
        "t": 12.04352333803672,
        "x": 317.2758841909354,
        "y": 0,
        "z": -45.63919913052327,
        "shotId": 84
      },
      {
        "id": 16595,
        "t": 12.0656233361964,
        "x": 317.2935610349226,
        "y": 0,
        "z": -45.64120609005967,
        "shotId": 84
      },
      {
        "id": 16596,
        "t": 12.08772333435608,
        "x": 317.3106767092595,
        "y": 0,
        "z": -45.64314933659492,
        "shotId": 84
      },
      {
        "id": 16597,
        "t": 12.10982333251576,
        "x": 317.3272312139459,
        "y": 0,
        "z": -45.64502887012901,
        "shotId": 84
      },
      {
        "id": 16598,
        "t": 12.13192333067544,
        "x": 317.3432245489821,
        "y": 0,
        "z": -45.64684469066194,
        "shotId": 84
      },
      {
        "id": 16599,
        "t": 12.15402332883512,
        "x": 317.3586567143677,
        "y": 0,
        "z": -45.64859679819372,
        "shotId": 84
      },
      {
        "id": 16600,
        "t": 12.1761233269948,
        "x": 317.373527710103,
        "y": 0,
        "z": -45.65028519272435,
        "shotId": 84
      },
      {
        "id": 16601,
        "t": 12.19822332515448,
        "x": 317.387837536188,
        "y": 0,
        "z": -45.65190987425381,
        "shotId": 84
      },
      {
        "id": 16602,
        "t": 12.22032332331416,
        "x": 317.4015861926225,
        "y": 0,
        "z": -45.65347084278212,
        "shotId": 84
      },
      {
        "id": 16603,
        "t": 12.24242332147384,
        "x": 317.4147736794066,
        "y": 0,
        "z": -45.65496809830928,
        "shotId": 84
      },
      {
        "id": 16604,
        "t": 12.26452331963352,
        "x": 317.4273999965404,
        "y": 0,
        "z": -45.65640164083528,
        "shotId": 84
      },
      {
        "id": 16605,
        "t": 12.2866233177932,
        "x": 317.4394651440238,
        "y": 0,
        "z": -45.65777147036012,
        "shotId": 84
      },
      {
        "id": 16606,
        "t": 12.30872331595288,
        "x": 317.4509691218568,
        "y": 0,
        "z": -45.65907758688382,
        "shotId": 84
      },
      {
        "id": 16607,
        "t": 12.33082331411256,
        "x": 317.4619119300393,
        "y": 0,
        "z": -45.66031999040634,
        "shotId": 84
      },
      {
        "id": 16608,
        "t": 12.35292331227224,
        "x": 317.4722935685715,
        "y": 0,
        "z": -45.66149868092773,
        "shotId": 84
      },
      {
        "id": 16609,
        "t": 12.37502331043192,
        "x": 317.4821140374532,
        "y": 0,
        "z": -45.66261365844795,
        "shotId": 84
      },
      {
        "id": 16610,
        "t": 12.3971233085916,
        "x": 317.4913733366848,
        "y": 0,
        "z": -45.66366492296701,
        "shotId": 84
      },
      {
        "id": 16611,
        "t": 12.41922330675128,
        "x": 317.5000714662657,
        "y": 0,
        "z": -45.66465247448493,
        "shotId": 84
      },
      {
        "id": 16612,
        "t": 12.44132330491096,
        "x": 317.5082084261963,
        "y": 0,
        "z": -45.66557631300168,
        "shotId": 84
      },
      {
        "id": 16613,
        "t": 12.46342330307064,
        "x": 317.5157842164766,
        "y": 0,
        "z": -45.66643643851728,
        "shotId": 84
      },
      {
        "id": 16614,
        "t": 12.48552330123032,
        "x": 317.5227988371065,
        "y": 0,
        "z": -45.66723285103173,
        "shotId": 84
      },
      {
        "id": 16615,
        "t": 12.50762329939,
        "x": 317.529252288086,
        "y": 0,
        "z": -45.66796555054502,
        "shotId": 84
      },
      {
        "id": 16616,
        "t": 12.52972329754968,
        "x": 317.535144569415,
        "y": 0,
        "z": -45.66863453705715,
        "shotId": 84
      },
      {
        "id": 16617,
        "t": 12.55182329570936,
        "x": 317.5404756810937,
        "y": 0,
        "z": -45.66923981056812,
        "shotId": 84
      },
      {
        "id": 16618,
        "t": 12.57392329386904,
        "x": 317.545245623122,
        "y": 0,
        "z": -45.66978137107795,
        "shotId": 84
      },
      {
        "id": 16619,
        "t": 12.59602329202872,
        "x": 317.5494543954999,
        "y": 0,
        "z": -45.67025921858662,
        "shotId": 84
      },
      {
        "id": 16620,
        "t": 12.6181232901884,
        "x": 317.5531019982275,
        "y": 0,
        "z": -45.67067335309413,
        "shotId": 84
      },
      {
        "id": 16621,
        "t": 12.64022328834808,
        "x": 317.5561884313046,
        "y": 0,
        "z": -45.67102377460049,
        "shotId": 84
      },
      {
        "id": 16622,
        "t": 12.66232328650776,
        "x": 317.5587136947314,
        "y": 0,
        "z": -45.67131048310569,
        "shotId": 84
      },
      {
        "id": 16623,
        "t": 12.68442328466744,
        "x": 317.5606777885077,
        "y": 0,
        "z": -45.67153347860973,
        "shotId": 84
      },
      {
        "id": 16624,
        "t": 12.70652328282712,
        "x": 317.5620807126336,
        "y": 0,
        "z": -45.67169276111262,
        "shotId": 84
      },
      {
        "id": 16625,
        "t": 12.7286232809868,
        "x": 317.5629224671093,
        "y": 0,
        "z": -45.67178833061435,
        "shotId": 84
      },
      {
        "id": 16626,
        "t": 12.75072327914648,
        "x": 317.5632030519345,
        "y": 0,
        "z": -45.67182018711492,
        "shotId": 84
      }
    ]
  }
]
Response Code
200
