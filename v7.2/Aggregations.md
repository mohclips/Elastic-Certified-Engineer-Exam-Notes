# Aggregations

# Write and execute metric and bucket aggregations

We will be using the `kibana_sample_data_ecommerce` index data for these examples.

Show only the aggs results, and not all of the matches: 
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/returning-only-agg-results.html


## Metric Aggregations

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html

### Common
- Avg Aggregation
- Max Aggregation
- Min Aggregation
- Sum Aggregation
- Stats Aggregation
- Extended Stats Aggregation

### Not so common
- Cardinality Aggregation
- Geo Bounds Aggregation
- Geo Centroid Aggregation
- Median Absolute Deviation Aggregation
- Percentiles Aggregation
- Percentile Ranks Aggregation
- Scripted Metric Aggregation
- Top Hits Aggregation
- Value Count Aggregation
- Weighted Avg Aggregation

<hr>

:question: Pull the number of sales, Max, Min, Average and total sales for the American customers.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics-extendedstats-aggregation.html

> Note: We use `"size": 0` to only return the aggs results and not all the actual matching documents.

> Note: We still need to use the `filter_path` to drill down to our aggs results.

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size": 0,
  "query": {
    "match": {
      "geoip.country_iso_code": "US"
    }
  },
  "aggs": {
    "cart_stats": {
      "extended_stats": {
        "field": "taxful_total_price"
      }
    }
  }
}

\\ output

{
  "aggregations" : {
    "cart_stats" : {
      "count" : 1206,
      "min" : 6.98828125,
      "max" : 344.0,
      "avg" : 76.16962064676616,
      "sum" : 91860.5625,
      "sum_of_squares" : 9216415.566009521,
      "variance" : 1840.3245174012998,
      "std_deviation" : 42.89900368774664,
      "std_deviation_bounds" : {
        "upper" : 161.96762802225945,
        "lower" : -9.628386728727122
      }
    }
  }
}
```
</details>
<hr>

## Bucket Aggregations

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html


- Adjacency Matrix Aggregation
- Auto-interval Date Histogram Aggregation
- Children Aggregation
- Composite Aggregation
- Date Histogram Aggregation
- Date Range Aggregation
- Diversified Sampler Aggregation
- Filter Aggregation
- Filters Aggregation
- Geo Distance Aggregation
- GeoHash grid Aggregation
- GeoTile Grid Aggregation
- Global Aggregation
- Histogram Aggregation
- IP Range Aggregation
- Missing Aggregation
- Nested Aggregation
- Parent Aggregation
- Range Aggregation
- Reverse nested Aggregation
- Sampler Aggregation
- Significant Terms Aggregation
- Significant Text Aggregation
- Terms Aggregation

:question: Display all sales per day

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      }
    }
  }
}

\\ output


{
  "aggregations" : {
    "date_hist" : {
      "buckets" : [
        {
          "key_as_string" : "2021-03-25T00:00:00.000Z",
          "key" : 1616630400000,
          "doc_count" : 146
        },
        {
          "key_as_string" : "2021-03-26T00:00:00.000Z",
          "key" : 1616716800000,
          "doc_count" : 153
        },
        {
          "key_as_string" : "2021-03-27T00:00:00.000Z",
          "key" : 1616803200000,
          "doc_count" : 143
        },
        ...
```
</details>
<hr>


# Write and execute aggregations that contain sub-aggregations


:question: Display all sales per day broken down by sales category

<details>
  <summary>View Solution (click to reveal)</summary>

> - The `date_histogram` is the Y-axis
> - Then you need to group by category
> - then Sum the sales (X-axis)

Create the `date_histogram`
```json 
todo?
```

Aggregate by the `category` grouping - here you are testing your aggregation.

You should also note the number of categories, so you don't `size` the aggregation too small later on.

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size": 0,
  "aggs": {
    "cat_keyword": {
      "terms": {
        "field": "category.keyword"
      }
    }
  }
}

// output
{
  "aggregations" : {
    "cat_keyword" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Men's Clothing",
          "doc_count" : 2024
        },
        {
          "key" : "Women's Clothing",
          "doc_count" : 1903
        },
        {
          "key" : "Women's Shoes",
          "doc_count" : 1136
        },
        {
          "key" : "Men's Shoes",
          "doc_count" : 944
        },
        {
          "key" : "Women's Accessories",
          "doc_count" : 830
        },
        {
          "key" : "Men's Accessories",
          "doc_count" : 572
        }
      ]
    }
  }
}

```

Put it all together

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist_y_axis": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      },
      "aggs": {
        "group_by_category": {
          "terms": {
            "field": "category.keyword",
            "size": 8
          },
          "aggs": {
            "total_sales_price": {
              "sum": {
                "field": "taxful_total_price"
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

BONUS QUESTION: Order the sales price in descending order

<details>
  <summary>View Solution (click to reveal)</summary>

In the `group_by_category` agg, we need to order by the `total_sales_price` agg.


```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist_y_axis": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      },
      "aggs": {
        "group_by_category": {
          "terms": {
            "field": "category.keyword",
            "order": {
              "total_sales_price": "desc"
            },
            "size": 8
          },
          "aggs": {
            "total_sales_price": {
              "sum": {
                "field": "taxful_total_price"
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

# Write and execute pipeline aggregations

:question: Work out the weekly sales maximum?

(Which week has the maximum sales?)

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "aggs": {
        "sales_per_week" : {
            "date_histogram" : {
                "field" : "order_date",
                "calendar_interval" : "1w"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "taxful_total_price"
                    }
                }
            }
        },
        "max_weekly_sales": {
            "max_bucket": {
                "buckets_path": "sales_per_week>sales" 
            }
        }
    }
}

// output

{
  "aggregations" : {
    "sales_per_week" : {
      "buckets" : [
        {
          "key_as_string" : "2021-03-22T00:00:00.000Z",
          "key" : 1616371200000,
          "doc_count" : 582,
          "sales" : {
            "value" : 41455.5390625
          }
        },
        {
          "key_as_string" : "2021-03-29T00:00:00.000Z",
          "key" : 1616976000000,
          "doc_count" : 1048,
          "sales" : {
            "value" : 79448.60546875
          }
        },
        {
          "key_as_string" : "2021-04-05T00:00:00.000Z",
          "key" : 1617580800000,
          "doc_count" : 1048,
          "sales" : {
            "value" : 78208.4296875
          }
        },
        {
          "key_as_string" : "2021-04-12T00:00:00.000Z",
          "key" : 1618185600000,
          "doc_count" : 1073,
          "sales" : {
            "value" : 81277.296875
          }
        },
        {
          "key_as_string" : "2021-04-19T00:00:00.000Z",
          "key" : 1618790400000,
          "doc_count" : 924,
          "sales" : {
            "value" : 70494.2578125
          }
        }
      ]
    },
    "max_weekly_sales" : {
      "value" : 81277.296875,
      "keys" : [
        "2021-04-12T00:00:00.000Z"
      ]
    }
  }
}

```

</details>
<hr>
