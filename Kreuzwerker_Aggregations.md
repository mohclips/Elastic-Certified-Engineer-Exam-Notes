# Kreuzwerker Aggregations

:bulb: See https://medium.com/kreuzwerker-gmbh/exercises-for-the-elastic-certified-engineer-exam-search-and-aggregations-1eefcfb6e992

These are very good.  My answers below.


# Questions and Answers

If you feel any of these are incorrect please raise an issue ticket. :)

```json
GET kibana_sample_data_flights
```

# Metrics

:question: Create an aggregation named "max_distance" that calculates the maximum value of the `DistanceKilometers` field

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "max_distance": {
      "max": {
        "field": "DistanceKilometers"
      }
    }
  }
}
```
</details>
<hr>

:question: Create an aggregation named "stats_flight_time" that computes stats over the value of the `FlightTimeMin` field

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "stats_flight_time": {
      "stats": {
        "field": "FlightTimeMin"
      }
    }
  }
}
```
</details>
<hr>

:question: Create two aggregations, named "cardinality_origin_cities" and "cardinality_dest_cities", that count the distinct values of the `OriginCityName` and `DestCityName` fields, respectively

<details>
  <summary>View Solution (click to reveal)</summary>

1.
```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "cardinality_origin_cities": {
      "cardinality": {
        "field": "OriginCityName"
      }
    }
  }
}
```

2.
```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "cardinality_dest_cities": {
      "cardinality": {
        "field": "DestCityName"
      }
    }
  }
}
```
</details>
<hr>


# Buckets


:question: Create an aggregation named "popular_origin_cities" that 
   calculates the number of flights grouped by the 
   `OriginCityName` field

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "popular_origin_cities": {
      "terms": {
        "field": "OriginCityName"
      }
    }
  }
}
```
</details>
<hr>

:question: As above, but return only 5 buckets and in descending order

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "popular_origin_cities": {
      "terms": {
        "field": "OriginCityName",
        "size": 5,
        "order" : { "_count" : "desc" }
      }
    }
  }
}
```
</details>
<hr>

:question: Create an aggregation named "avg_price_histogram" that groups the 
   documents based on `AvgTicketPrice` by intervals of 250

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "avg_price_histogram": {
      "histogram": {
        "field": "AvgTicketPrice",
        "interval": 250
      }
    }
  }
}
```
</details>
<hr>

:question: Create an aggregation named "popular_carriers" that calculates the 
   number of flights grouped by the `Carrier` field

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "popular_carriers": {
      "terms": {
        "field": "Carrier"
      }
    }
  }
}
```
</details>
<hr>

:question: Add a sub-aggregation to "popular_carriers", named 
   "carrier_stats_delay", that computes stats over the value of the 
   `FlightDelayMin` field for the related bucket of carriers

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0, 
  "aggs": {
    "popular_carriers": {
      "terms": {
        "field": "Carrier"
      },
      "aggs": {
        "carrier_stats_delay": {
          "stats": {
            "field": "FlightDelayMin"
          }
        }
      }
    }
  }
}
```
</details>
<hr>

:question: Add a second sub-aggregation to "popular_carriers", named 
   "carrier_max_delay", that shows the flight having the maximum 
   value of the `FlightDelayMin` field for the related bucket of 
   carriers

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "popular_carriers": {
      "terms": {
        "field": "Carrier"
      },
      "aggs": {
        "carrier_stats_delay": {
          "stats": {
            "field": "FlightDelayMin"
          }
        },
        "carrier_max_delay": {
          "max": {
            "field": "FlightDelayMin"
          }
        }
      }
    }
  }
}
```
</details>
<hr>

# date histograms


:question: Use the `timestamp` field to create an aggregation named 
   "flights_every_10_days" that groups the number of flights by an 
   interval of 10 days

<details>
  <summary>View Solution (click to reveal)</summary>

```json
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_every_10_days": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "10d"
      }
    }
  }
}
```


## wut!
<pre>
  "type" : "x_content_parse_exception",
  "reason" : "[7:39] [date_histogram] failed to parse field [calendar_interval]",
</pre>

> If you attempt to use multiples of calendar units, the aggregation will fail because only singular calendar units are supported:

retry, with the old style `interval`

tah dah!

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_every_10_days": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "10d"
      }
    }
  }
}
```
</details>
<hr>

:question: Use the `timestamp` field to create an aggregation named 
   "flights_by_day" that groups  the number of flights by day 

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      }
    }
  }
}
```
</details>
<hr>

:question: Add a sub-aggregation to “flights_by_day”, named 
   “destinations_by_day”, that groups the day buckets by the value 
   of the `DestCityName` field

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        }
      }
    }
  }
}
```
</details>
<hr>

:question: Add a sub-aggregation to the sub-aggregation 
   "destinations_by_day", named "popular_destinations_by_day", that 
   returns the 3 most popular documents for each bucket (i.e., 
   ordered by their score)

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {
                "size": 3
              }
            }
          }
        }
      }
    }
  }
}
```
</details>
<hr>

:question: Update “popular_destinations_by_day” to display only the 
   `SourceCityName` field in for each top hit object

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {
                "size": 3,
                "_source": {
                  "includes": [
                    "OriginCityName"
                  ]
                }
              }
            }
          }
        }
      }
    }
  }
}
```
</details>
<hr>

# pipelines


:question: Remove the "popular_destinations_by_day” sub-sub-aggregation from 
   “flights_by_day”

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        }
      }
    }
  }
}
```
</details>
<hr>

:question: Add a pipeline aggregation to "flights_by_day", named 
   "most_popular_destination_of_the_day", that identifies the 
   "popular_destinations_by_day” bucket with the most documents for 
   each day

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "dest_counter": {
              "value_count": {
                "field": "_id"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>dest_counter.value"
          }
        }
      }
    }
  }
}
```
</details>
<hr>


:bulb: with `order:desc` and `size:1` - just so you can see the right value in the destinations_by_day buckets

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName",
            "size": 1,
            "order": {
              "_count": "desc"
            }
          },
          "aggs": {
            "dest_counter": {
              "value_count": {
                "field": "_id"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>dest_counter.value"
          }
        }
      }
    }
  }
}
```
</details>
<hr>

:question: Add a pipeline aggregation named "day_with_most_flights" that 
   identifies the “flights_by_day” bucket with the most documents

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName",
            "size": 1,
            "order": {
              "_count": "desc"
            }
          },
          "aggs": {
            "dest_counter": {
              "value_count": {
                "field": "_id"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>dest_counter.value"
          }
        }
      }
    },
    "day_with_most_flights" : {
      "max_bucket" : {
        "buckets_path": "flights_by_day._count"
      }
    }
  }
}
```
</details>
<hr>

:question: Add a pipeline aggregation named 
   "day_with_the_most_popular_destination_over_all_days" that 
   identifies the “flights_by_day” bucket with the largest 
   “most_popular_destination_of_the_day” value

<details>
  <summary>View Solution (click to reveal)</summary>
  
```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName",
            "size": 1,
            "order": {
              "_count": "desc"
            }
          },
          "aggs": {
            "dest_counter": {
              "value_count": {
                "field": "_id"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>dest_counter.value"
          }
        }
      }
    },
    "day_with_most_flights" : {
      "max_bucket" : {
        "buckets_path": "flights_by_day._count"
      }
    },
    "day_with_the_most_popular_destination_over_all_days" : {
      "max_bucket": {
        "buckets_path": "flights_by_day>most_popular_destination_of_the_day"
      }
    }
  }
}
```
</details>
<hr>

### Fin!