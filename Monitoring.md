# Grafana

https://monitor.lichess.org/

## Important dashboards

* [HTTP/WS](https://monitor.lichess.org/dashboard/db/lichess-http-ws)
* [JVM](https://monitor.lichess.org/dashboard/db/lichess-jvm)
* [Gameplay](https://monitor.lichess.org/dashboard/db/lichess-gameplay)
* [Lobby](https://monitor.lichess.org/dashboard/db/lichess-lobby)

## Monitoring stack

Every server has a Telegraf daemon collecting and sending relevant data to a central Influxdb database. Likewise, lila uses Kamon to send application data to the Influxdb server.

Grafana connects to Influxdb to display graphs.

The "historical" stack was collectd and Graphite-API, plus Kamon and statsd for lila, it is slowly being replaced by the Telegraf/Influxdb stack.