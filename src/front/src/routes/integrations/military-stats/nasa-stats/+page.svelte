<script>
    import { onMount } from "svelte";
    import Highcharts from "highcharts";

    onMount(async () => {
        // 1. Fetch a tu PROXY propio (Requisito obligatorio)
        const response = await fetch("/api/v2/military-stats/proxy/nasa-stats");
        const json = await response.json();
        
        // 2. Procesar datos (Asteroides de una fecha específica)
        const date = "2026-05-04";
        const asteroids = json.near_earth_objects[date] || [];

        const seriesData = asteroids.map(a => ({
            name: a.name,
            y: a.estimated_diameter.meters.estimated_diameter_max
        }));

        // 3. Highcharts - Tipo 'bar' (para variar de 'column')
        Highcharts.chart("nasa-chart", {
            chart: { type: 'bar' },
            title: { text: 'Tamaño de Asteroides Cercanos (Vía Proxy)' },
            xAxis: { type: 'category' },
            yAxis: { title: { text: 'Diámetro Máximo (m)' } },
            series: [{
                name: 'Metros',
                data: seriesData,
                color: '#2c3e50'
            }]
        });
    });
</script>

<div id="nasa-chart"></div>