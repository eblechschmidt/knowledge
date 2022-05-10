# Retrieve displayable table for mysql replication status

The following flux script returns a table containing the columns `Channel_Name`, `Master_Host`, `host` and `Slave_IO_Running` from an influxdb that were retireved and stored from a mysql db using telegraf.

```flux
from(bucket: "monitoring")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "slave_status")
  |> filter(fn: (r) => r["_field"] == "Slave_IO_Running")
  |> group(columns: ["Channel_Name"]) 
  |> last()
  |> group()
  |> keep(columns: ["_value", "Channel_Name", "host", "Master_Host"])
  |> rename(columns: {_value: "Slave_IO_Running"})
```

  #telegraf #influxdb #mysql #monitoring
