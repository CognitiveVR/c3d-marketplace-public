Full reference guide for slicer request aggregation output formats. The dimensionality of your data plays a big role here. An aggregation with no slice by will return 0 dimensional data (so just a single number), 1 slice by will give you 1 dimension, etc.

This will cover, for 0,1,2+ dimensions, the current 4 output formats available. For each of these, there will be an example output and the exact input used to get that output. You could paste it into postman, get the same data, and play around with it to see how it works.

As these are just formatting options, they don't affect the data itself, just how it gets formatted. This is particularly noticeable here as in each dimension # the same request is used for all 4 output formats examples. Only the output format changes in the input example JSONs.

# 0 Dimensions
The simplest case. The output for 0 dimensions is fundamentally just a single number.

Let's use as an example a request to get the total session time for our project.

### Output Type "legacy"

The value is in the key `value`. Surprise! There's some misc other fields you get here that were (still are? IDK) used to control axis labels and charting options remotely on the analysis page.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "count_total_session_time": {
          "dimensionless": true,
          "label": "count_total_session_time",
          "value": 81214962
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 541
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "legacy",
        "operations": [
          {
            "name": "count_total_session_time",
            "type": "sum",
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          }
        ]
      }
    ]
  }
  ```
</details>

### Output Type "json0_keyed" or "json0_list" or "json0_key_y"

All 3 of the non-legacy formats are the same here. The legacy charting fields are no longer present. You get just the number in the field `value` as with the legacy format, along with a warnings array used for debugging. (This debugging array will be seen in all subsequent non-legacy formats.)

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "count_total_session_time": {
          "value": 81214962,
          "warnings": []
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 541
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_keyed",
        "operations": [
          {
            "name": "count_total_session_time",
            "type": "sum",
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          }
        ]
      }
    ]
  }
  ```
</details>

# 1 Dimension
With 1 dimension, the output is a value per bucket and each bucket is defined by the single axis of the data.

Let's make a request that aggregates the average app performance by session duration. We'll produce a histogram with 10-minute intervals of session duration. That's the `600000` (10 minutes in milliseconds) you will see in the inputs.

> **Note:** these examples aggregate `c3d.metrics.app_performance`, which is now deprecated — sessions from current SDK/backend versions store a constant `50` in it (you can spot one such bucket in the `json0_keyed` example below). The examples are kept because they teach the output *shapes*, which are unchanged; for real queries substitute `c3d.metrics.fps_score` (present on recent sessions; sessions predating its rollout may lack it).

### Output Type "legacy"

The legacy format's charting control fields now includes "xIsTimeseries" now that we have an axis; this was used to switch the chart between line and bar (or pie?) charts. The data comes in a list of objects with an x value and a y value; the y value is an array always with exactly 1 item in it for 1 dimensional data. The x value is the value of the bucket defined by your 1 slice by and the y value is the value of your aggregation's operation in that bucket.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "average_app_performance": {
          "dimensionless": false,
          "xIsTimeseries": false,
          "labels": [
            "average_app_performance"
          ],
          "values": [
            {
              "x": 0,
              "y": [
                57.627841876103325
              ]
            },
            {
              "x": 600000,
              "y": [
                57.50388145446777
              ]
            },
            {
              "x": 1200000,
              "y": [
                57.47849464416504
              ]
            },
            {
              "x": 1800000,
              "y": [
                50
              ]
            },
            {
              "x": 2400000,
              "y": [
                null
              ]
            },
            {
              "x": 3000000,
              "y": [
                null
              ]
            },
            {
              "x": 3600000,
              "y": [
                80.00086975097656
              ]
            },
            {
              "x": 4200000,
              "y": [
                null
              ]
            },
            {
              "x": 4800000,
              "y": [
                null
              ]
            },
            {
              "x": 5400000,
              "y": [
                null
              ]
            },
            {
              "x": 6000000,
              "y": [
                null
              ]
            },
            {
              "x": 6600000,
              "y": [
                null
              ]
            },
            {
              "x": 7200000,
              "y": [
                80.00502395629883
              ]
            }
          ]
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 541
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "legacy",
        "sliceBys": [
          {
            "name": "slice_by_session_duration",
            "field": {
              "fieldName": "duration",
              "fieldParent": "session"
            },
            "interval": 600000
          }
        ],
        "operations": [
          {
            "name": "average_app_performance",
            "type": "average",
            "field": {
              "unnestedFieldName": "numericalSessionProp",
              "path": "c3d.metrics.app_performance"
            }
          }
        ]
      }
    ]
  }
  ```
</details>

### Output Type "json0_keyed"

With this output type, the data comes in an object. The keys are the values of the buckets defined by your 1 slice by and the value at each key is the value of your aggregation's operation within that bucket. For histogram and date histogram slice by's, empty buckets that are between nonempty buckets get filled in (either with null or 0 depending on the operation, in this case because we are performing an average, we get nulls).

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "average_app_performance": {
          "values": {
            "0.0": 57.627841876103325,
            "600000.0": 57.50388145446777,
            "1200000.0": 57.47849464416504,
            "1800000.0": 50,
            "2400000.0": null,
            "3000000.0": null,
            "3600000.0": 80.00086975097656,
            "4200000.0": null,
            "4800000.0": null,
            "5400000.0": null,
            "6000000.0": null,
            "6600000.0": null,
            "7200000.0": 80.00502395629883
          },
          "warnings": []
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 541
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_keyed",
        "sliceBys": [
          {
            "name": "slice_by_session_duration",
            "field": {
              "fieldName": "duration",
              "fieldParent": "session"
            },
            "interval": 600000
          }
        ],
        "operations": [
          {
            "name": "average_app_performance",
            "type": "average",
            "field": {
              "unnestedFieldName": "numericalSessionProp",
              "path": "c3d.metrics.app_performance"
            }
          }
        ]
      }
    ]
  }
  ```
</details>

### Output Type "json0_list" or "json0_key_y"

These 2 output types give identical outputs when you have 1 dimension. You get an array of values. Each object in the array represents 1 bucket; the x value is the value of the bucket defined by your 1 slice by and the value is the value of your aggregation's operation within that bucket.

For histogram or date histogram slice by's, empty buckets get included similarly to the json0_keyed output type. This is unintentional; the "json0_list" type is supposed to always skip empty buckets and only the "json0_key_y" type will include them when this is fixed.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "average_app_performance": {
          "values": [
            {
              "x": 0,
              "value": 57.627841876103325
            },
            {
              "x": 600000,
              "value": 57.50388145446777
            },
            {
              "x": 1200000,
              "value": 57.47849464416504
            },
            {
              "x": 1800000,
              "value": 50
            },
            {
              "x": 2400000,
              "value": null
            },
            {
              "x": 3000000,
              "value": null
            },
            {
              "x": 3600000,
              "value": 80.00086975097656
            },
            {
              "x": 4200000,
              "value": null
            },
            {
              "x": 4800000,
              "value": null
            },
            {
              "x": 5400000,
              "value": null
            },
            {
              "x": 6000000,
              "value": null
            },
            {
              "x": 6600000,
              "value": null
            },
            {
              "x": 7200000,
              "value": 80.00502395629883
            }
          ],
          "warnings": []
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
    {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 541
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_list",
        "sliceBys": [
          {
            "name": "slice_by_session_duration",
            "field": {
              "fieldName": "duration",
              "fieldParent": "session"
            },
            "interval": 600000
          }
        ],
        "operations": [
          {
            "name": "average_app_performance",
            "type": "average",
            "field": {
              "unnestedFieldName": "numericalSessionProp",
              "path": "c3d.metrics.app_performance"
            }
          }
        ]
      }
    ]
  }
  ```
</details>

# 2 Dimensions
With 2 dimensions, the output is still a value per bucket, but buckets are defined by overlapping values of 2 axes.

Let's make a request that counts sessions grouped by duration again and ergonomics score.

This is similar the prior example, but fixes a fundamental problem there: we had the average value of our metric per bucket, but we didn't know the sample size of sessions that went in to that average in each of those buckets. So we could chart it, but it's hard to draw conclusions from. Here we instead move the metric from the operation as an average to just another dimension in the slice by's list, and do a session count operation instead. This way we will get a relationship between the two things we care about with a lot more confidence.

(In this case, the metric in question is the ergonomics score, prior example was of course app performance. Mixing it up for fun.)

In this example let's make the input extremely non-granular just so the output isn't huge. But by increasing the granularity (do 1 point chunks for ergo score instead of 25, and 1 minute intervals for duration instead of 10), you could get data to fuel a scatter plot with this request.

### Output Type "legacy"

The last slice by is called the x axis here, and the first is called the y axis. We first get a reference array `labels` of all possible values of our y axis in some order. Our data comes in an array where each object represents actually multiple buckets. The x value is the value of the x axis and then there is an array of y values. This array is as long as `labels`. You must cross-reference to `labels` to interpret it; it has a value here for each known possible value of the y axis. (If no data is present for _this_ particular combination of x and y, you will still have an entry in the JSON: it will be either 0 or null depending on the type of operation).

So in this case, the first item of each y array is the count of sessions at the given x value for the bucket "ergonomics score in range 50-75". (Histogram ranges are referred to by their start. So it's 50-75 beacuse the first item in labels is "50".)

The x axis also gets filled in for histogrma and date histogram slice by's in a similar way to described before: if no buckets corresponding to an x value would have data, typically that x value will just not show up at all, but for histograms and date histograms if that x value is between two or more other x values that do have data, it will be included with appropriate values (0 or null) for each y value.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "total_session_count": {
          "dimensionless": false,
          "xIsTimeseries": false,
          "labels": [
            "50",
            "100",
            "25",
            "0",
            "75"
          ],
          "values": [
            {
              "x": 0,
              "y": [
                40,
                83,
                32,
                87,
                37
              ]
            },
            {
              "x": 600000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 1200000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 1800000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 2400000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 3000000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 3600000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 4200000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 4800000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 5400000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 6000000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 6600000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 7200000,
              "y": [
                0,
                0,
                0,
                0,
                0
              ]
            },
            {
              "x": 7800000,
              "y": [
                0,
                1,
                0,
                0,
                0
              ]
            }
          ]
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 11
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "legacy",
        "sliceBys": [
          {
            "name": "slice by duration - 10 minute chunks",
            "interval": 600000,
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          },
          {
            "name": "slice by ergonomics score - 25 point chunks",
            "interval": 25,
            "field": {
              "nestedFieldName": "numericalSessionProp",
              "fieldParent": "session",
              "path": "c3d.metrics.ergonomics_score"
            }
          }
        ],
        "operations": [
          {
            "name": "total_session_count",
            "type": "sessionCount"
          }
        ]
      }
    ]
  }
  ```
</details>

### Output Type "json0_keyed"

We get a nested object. The keys of the outermost and then its inner object together define each bucket. The value at this path is the value of the operation in that bucket. The outer key will be the value of the first slice by and the inner key will be the value of the second slice by.

Compared to legacy, this aggregation type skips empty buckets in the second dimension but not the first. This "filling in" in the first dimension still occurs in histograms and date histograms as described before. In this particular example, the bucket of session durations starting at "0.0" has values and the bucket of durations starting at "7800000.0" has values so all the intermediate session duration buckets between the specific "0-10 minute" bucket and the "130-140 minute" bucket get filled in even though they are all empty.

(As mentioned before, buckets are referenced by their start point, so because we have a 10 minute interval we know "0.0" means 0-10 minutes.)

<details>
  <summary>Output Example</summary>

  ```json
{
  "aggregations": {
    "main": {
      "total_session_count": {
        "values": {
          "1200000.0": {},
          "5400000.0": {},
          "6600000.0": {},
          "2400000.0": {},
          "3600000.0": {},
          "4800000.0": {},
          "7800000.0": {
            "100": 1
          },
          "1800000.0": {},
          "0.0": {
            "100": 83,
            "50": 40,
            "25": 32,
            "75": 37,
            "0": 87
          },
          "6000000.0": {},
          "600000.0": {},
          "3000000.0": {},
          "7200000.0": {},
          "4200000.0": {}
        },
        "warnings": []
      }
    }
  }
}
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 11
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_keyed",
        "sliceBys": [
          {
            "name": "slice by duration - 10 minute chunks",
            "interval": 600000,
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          },
          {
            "name": "slice by ergonomics score - 25 point chunks",
            "interval": 25,
            "field": {
              "nestedFieldName": "numericalSessionProp",
              "fieldParent": "session",
              "path": "c3d.metrics.ergonomics_score"
            }
          }
        ],
        "operations": [
          {
            "name": "total_session_count",
            "type": "sessionCount"
          }
        ]
      }
    ]
  }
  ```
</details>


> This is visible in all of these 2 dimension outputs, but I'll make an aside now to mention: in practice, it's very clear with this request that we need to up our granularity to dig into the bulk of our data which lives in the 0-10 minute range, and perhaps filter out that singular data point in the 130min+ range as an outlier.


### Output Type "json0_list"

We get an array where each object represents a single bucket of our data. Each has 3 fields: x, y, value. The value is the value of the operation in the bucket defined by our 2 slice by's; the value of the first is x and value of the second is y.

Compared to the previous two types, only the exact buckets where data is defined at all are given here. If there is no data for a particular combination of x and y, it will not be present in the JSON at all no matter what. This makes it the most compact format available.

This is also probably unergonomic for charting on the FE in most cases because you may have to recreate those missing buckets depending on how you want to chart this data. Would be good for a scatter plot, maybe bad for a line chart.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "total_session_count": {
          "values": [
            {
              "x": 0,
              "y": "50",
              "value": 40
            },
            {
              "x": 0,
              "y": "100",
              "value": 83
            },
            {
              "x": 0,
              "y": "25",
              "value": 32
            },
            {
              "x": 0,
              "y": "0",
              "value": 87
            },
            {
              "x": 0,
              "y": "75",
              "value": 37
            },
            {
              "x": 7800000,
              "y": "100",
              "value": 1
            }
          ],
          "warnings": []
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 11
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_list",
        "sliceBys": [
          {
            "name": "slice by duration - 10 minute chunks",
            "interval": 600000,
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          },
          {
            "name": "slice by ergonomics score - 25 point chunks",
            "interval": 25,
            "field": {
              "nestedFieldName": "numericalSessionProp",
              "fieldParent": "session",
              "path": "c3d.metrics.ergonomics_score"
            }
          }
        ],
        "operations": [
          {
            "name": "total_session_count",
            "type": "sessionCount"
          }
        ]
      }
    ]
  }
  ```
</details>


### Output Type "json0_key_y"

This output type closely mirrors the structure of the legacy format. The charting control stuff is removed of course, and instead of the labels array you just get the values keyed by the bucket's y value in each JSON object.

In this output type both axes are fully expanded. The x axis will get missing values filled in for histogram and date histograms. The y axis will list values for all possible y values at each x value, with appropriate 0/null values for missing data.

<details>
  <summary>Output Example</summary>

  ```json
  {
    "aggregations": {
      "main": {
        "total_session_count": {
          "values": [
            {
              "x": 0,
              "values": {
                "0": 87,
                "100": 83,
                "25": 32,
                "50": 40,
                "75": 37
              }
            },
            {
              "x": 600000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 1200000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 1800000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 2400000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 3000000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 3600000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 4200000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 4800000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 5400000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 6000000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 6600000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 7200000,
              "values": {
                "0": 0,
                "100": 0,
                "25": 0,
                "50": 0,
                "75": 0
              }
            },
            {
              "x": 7800000,
              "values": {
                "0": 0,
                "100": 1,
                "25": 0,
                "50": 0,
                "75": 0
              }
            }
          ],
          "warnings": []
        }
      }
    }
  }
  ```
</details>
<details>
  <summary>Input Example</summary>

  ```json
  {
    "sessionType": "project",
    "entityFilters": {
      "projectId": 11
    },
    "aggregations": [
      {
        "name": "main",
        "outputType": "json0_key_y",
        "sliceBys": [
          {
            "name": "slice by duration - 10 minute chunks",
            "interval": 600000,
            "field": {
              "fieldParent": "session",
              "fieldName": "duration"
            }
          },
          {
            "name": "slice by ergonomics score - 25 point chunks",
            "interval": 25,
            "field": {
              "nestedFieldName": "numericalSessionProp",
              "fieldParent": "session",
              "path": "c3d.metrics.ergonomics_score"
            }
          }
        ],
        "operations": [
          {
            "name": "total_session_count",
            "type": "sessionCount"
          }
        ]
      }
    ]
  }
  ```
</details>
