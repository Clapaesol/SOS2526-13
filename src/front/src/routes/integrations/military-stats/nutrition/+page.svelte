<script>
    import { onMount } from "svelte";
    import Highcharts from "highcharts";

    let dataNutricion = [];
    let errorMsg = "";

    onMount(async () => {
        try {
            // 1. Llamada a tu proxy (BASE_API_URL definido en tu backend)
            const response = await fetch("/api/v2/military-stats/proxy/nutrition");
            
            if (!response.ok) {
                errorMsg = "Error al cargar los datos del proxy";
                return;
            }

            dataNutricion = await response.json();

            // 2. Preparar los datos para la gráfica
            // La API devuelve un array, cogemos el primer elemento (ej: cheeseburger)
            const alimento = dataNutricion[0];
            
            const stats = [
                { name: "Proteínas", y: alimento.protein_g },
                { name: "Grasas Totales", y: alimento.fat_total_g },
                { name: "Carbohidratos", y: alimento.carbohydrates_total_g },
                { name: "Azúcar", y: alimento.sugar_g },
                { name: "Fibra", y: alimento.fiber_g }
            ];

            // 3. Renderizar Highcharts
            Highcharts.chart("nutrition-container", {
                chart: {
                    type: 'column'
                },
                title: {
                    text: `Composición Nutricional: ${alimento.name.toUpperCase()}`
                },
                subtitle: {
                    text: `Datos obtenidos vía Proxy de RapidAPI`
                },
                xAxis: {
                    type: 'category',
                    title: { text: 'Nutrientes' }
                },
                yAxis: {
                    title: { text: 'Gramos (g)' }
                },
                legend: { enabled: false },
                series: [{
                    name: 'Gramos',
                    colorByPoint: true,
                    data: stats
                }]
            });

        } catch (error) {
            console.error("Error en la integración:", error);
            errorMsg = "No se pudo conectar con el servidor.";
        }
    });
</script>

<main>
    {#if errorMsg}
        <p style="color: red;">{errorMsg}</p>
    {/if}
    
    <div id="nutrition-container"></div>

    <div class="info">
        <p>Esta integración usa un <b>Proxy intermedio</b> programado en el backend para consultar datos de nutrición de forma segura.</p>
    </div>
</main>

<style>
    #nutrition-container {
        width: 100%;
        height: 450px;
        margin: 20px 0;
    }
    .info {
        background-color: #f9f9f9;
        padding: 1rem;
        border-left: 5px solid #2c3e50;
    }
</style>