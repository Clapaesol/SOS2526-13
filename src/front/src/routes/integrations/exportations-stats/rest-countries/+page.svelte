<script>
import { onMount } from "svelte";
import { browser } from "$app/environment";

function loadScript(src) {

    return new Promise(resolve => {

        if (document.querySelector(`script[src="${src}"]`)) {
            return resolve();
        }

        const s = document.createElement("script");

        s.src = src;
        s.onload = resolve;

        document.head.appendChild(s);
    });
}

onMount(async () => {

    if (!browser) return;

    await loadScript("https://code.highcharts.com/highcharts.js");

    await loadScript(
        "https://code.highcharts.com/modules/sunburst.js"
    );

    const res = await fetch(
        "https://restcountries.com/v3.1/region/europe?fields=name,population,region"
    );

    const countries = await res.json();

    const data = [];

    data.push({
        id: "Europe",
        parent: "",
        name: "Europe"
    });

    countries.forEach((country, i) => {

        data.push({
            id: `country-${i}`,
            parent: "Europe",
            name: country.name.common,
            value: country.population || 1
        });
    });

    Highcharts.chart("sunburst-chart", {

        chart: {
            backgroundColor: "#0f172a"
        },

        title: {
            text: "Población por pais en Europa",
            style: {
                color: "#ffffff"
            }
        },

        series: [{
            type: "sunburst",
            data,
            allowDrillToNode: true,
            cursor: "pointer",

            dataLabels: {
                format: "{point.name}",
                style: {
                    color: "#ffffff"
                }
            }
        }]
    });
});
</script>

<div
    id="sunburst-chart"
    style="height:700px;width:100%;"
></div>