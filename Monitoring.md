# Grafana

https://monitor.lichess.ovh/

## Monitoring stack

Every server has a Telegraf daemon collecting and sending relevant data to a central InfluxDB. Likewise, lila uses Kamon to send application data to the InfluxDB server.

Grafana connects to InfluxDB to display graphs.

The "historical" stack was collectd and Graphite-API, plus Kamon and statsd for lila. It has been replaced by the Telegraf/InfluxDB stack.