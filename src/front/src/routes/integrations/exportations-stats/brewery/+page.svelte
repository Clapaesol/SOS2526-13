<script>
import { onMount } from "svelte";
import { browser } from "$app/environment";

function loadScript(src) {
	return new Promise(resolve => {

		if (document.querySelector(`script[src="${src}"]`)) return resolve();

		const s = document.createElement("script");
		s.src = src;
		s.onload = resolve;
		document.head.appendChild(s);
	});
}

onMount(async () => {

	if (!browser) return;

	await loadScript("https://code.highcharts.com/highcharts.js");
	await loadScript("https://code.highcharts.com/modules/treemap.js");

	const res = await fetch(
		"/api/v2/exportations-stats/proxy/brewery"
	);

	const data = await res.json();

	const grouped = {};

	data.forEach(b => {

		const country = b.country || "Unknown";
		const state = b.state || "Unknown";

		if (!grouped[country]) grouped[country] = {};

		grouped[country][state] =
			(grouped[country][state] || 0) + 1;
	});

	const chartData = [];

	Object.entries(grouped).forEach(([country, states]) => {

		Object.entries(states).forEach(([state, value]) => {

			chartData.push({
				name: state,
				parent: country,
				value
			});
		});
	});

	Highcharts.chart("brewery", {

		chart: {
			backgroundColor: "#0f172a"
		},

		title: {
			text: "Distribución de cervecerías",
			style: {
				color: "#fff"
			}
		},

		colorAxis: {
			min: 0,
			stops: [
				[0, "#93c5fd"],
				[0.5, "#3b82f6"],
				[1, "#1e3a8a"]
			]
		},

		tooltip: {
			pointFormat:
				"<b>{point.name}</b><br>cervecerías: {point.value}"
		},

		series: [{
			type: "treemap",
			layoutAlgorithm: "squarified",
			data: chartData,
			dataLabels: {
				enabled: true,
				style: {
					color: "#fff",
					textOutline: "none"
				}
			}
		}]
	});
});
</script>

<div
	id="brewery"
	style="height: 650px; border-radius: 12px;"
></div>  
