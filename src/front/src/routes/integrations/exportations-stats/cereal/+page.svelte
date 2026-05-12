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
    await loadScript("https://code.highcharts.com/modules/funnel.js");

    const res = await fetch(
        "https://sos2526-18-cereal-productions-stable.onrender.com/api/v2/cereal-productions"
    );

    const data = await res.json();

    const grouped = {};

    data.forEach(d => {
        const country = d.country || "Unknown";

        grouped[country] =
            (grouped[country] || 0) +
            Number(d.cereal_production || 0);
    });

    const top = Object.entries(grouped)
        .map(([country, value]) => ({ country, value }))
        .sort((a, b) => b.value - a.value);

    Highcharts.chart("cereal-chart", {

        chart: {
            type: "funnel",
            backgroundColor: "#0f172a"
        },

        title: {
            text: "Top Cereal Production Funnel",
            style: { color: "#fff" }
        },

        plotOptions: {
            series: {
                dataLabels: {
                    enabled: true,
                    format: "<b>{point.name}</b>: {point.y}",
                    style: {
                        color: "#fff",
                        textOutline: "none"
                    }
                }
            }
        },

        series: [{
            name: "Cereal Production",
            data: top.map(d => [d.country, d.value])
        }]
    });
});
</script>

<div id="cereal-chart" style="height: 700px;"></div>