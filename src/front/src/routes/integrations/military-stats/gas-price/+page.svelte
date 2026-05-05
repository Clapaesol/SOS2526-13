
<script>
    import { onMount } from "svelte";
    import Highcharts from "highcharts";

    let errorMsg = "";

    onMount(async () => {
        try {
            // Llamamos a tu proxy
            const response = await fetch("/api/v2/military-stats/proxy/gas-price");
            
            if (!response.ok) {
                errorMsg = "Error al obtener datos del proxy. ¿Has puesto la API Key?";
                return;
            }

            const result = await response.json();
            
            // Los datos vienen en result.result (según la documentación de esa API)
            const gasData = result.result;

            // Preparamos los datos para la gráfica
            const chartData = [
                { name: "Gasolina (Regular)", y: parseFloat(gasData.gasoline) },
                { name: "Midgrade", y: parseFloat(gasData.midgrade) },
                { name: "Premium", y: parseFloat(gasData.premium) },
                { name: "Diesel", y: parseFloat(gasData.diesel) }
            ];

            Highcharts.chart("gas-chart", {
                chart: { type: 'column' },
                title: { text: 'Precios Medios del Combustible en EE. UU.' },
                subtitle: { text: 'Datos obtenidos vía Proxy Propio (RapidAPI)' },
                xAxis: { type: 'category' },
                yAxis: {
                    title: { text: 'Precio (USD)' }
                },
                legend: { enabled: false },
                series: [{
                    name: 'Precio',
                    colorByPoint: true,
                    data: chartData
                }]
            });

        } catch (error) {
            console.error(error);
            errorMsg = "Error de conexión con el servidor.";
        }
    });
</script>

<main>
    {#if errorMsg}
        <p style="color: red; font-weight: bold; text-align: center;">{errorMsg}</p>
    {/if}
    <div id="gas-chart"></div>
</main>

<style>
    #gas-chart {
        width: 100%;
        max-width: 800px;
        height: 450px;
        margin: 0 auto;
    }
</style>