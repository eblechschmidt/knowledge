# Installing influxdb as a service on windows

This is not working!!

- Set the environment variable `INFLUXD_CONFIG_PATH` to point to the right
  folder (e.g. `C:\\monitoring\\influxdb`). It seems that escaping the
  backslash is needed,

- Log out and back in for changes of environment variable to be active

- Add at least the following lines to your `config.toml`
  
  ```
  bolt-path = "C:\\monitoring\\influxdb\\data\\influxd.bolt"
  sqlite-path = "C:\\monitoring\\influxdb\\data\\influxd.sqlite"
  engine-path = "C:\\monitoring\\influxdb\\data\\engine"
  ```

  Make sure that the backslashes are escaped!

- In a cmd run:
  ```
  sc create influxdb binPath= "START /B c:\monitoring\influxdb\influxd.exe > c:\monitoring\influxdb\influxdb.log"
  ```
