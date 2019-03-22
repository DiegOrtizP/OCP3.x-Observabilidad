# Kibana

Elasticsearch, su aplicación de visualización complementaria Kibana, y mostraré cómo .Net Core puede integrarse fácilmente con la pila Elastic.

## Para maquinas Windows



```csharp
private static readonly Histogram OrderValueHistogram = Metrics
	.CreateHistogram("myapp_order_value_usd", "Histogram of received order values (in USD).",
		new HistogramConfiguration
		{
			// We divide measurements in 10 buckets of $100 each, up to $1000.
			Buckets = Histogram.LinearBuckets(start: 100, width: 100, count: 10)
		});
```

![Alt text](relative/path/to/img.jpg?raw=true "Title")

`kibana`
