<script>
    import { onMount } from "svelte";

    let data = [];
    let Highcharts;

    function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement("script");
            script.src = src;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    onMount(async () => {
        try {
            await loadScript("https://code.highcharts.com/highcharts.js");
            Highcharts = window.Highcharts;

            const res = await fetch("/api/proxy/cereal");
            data = await res.json();

            const chartData = data.map(d => ({
                name: d.country,
                y: Number(d.cereal_production)
            }));

            Highcharts.chart("cereal-chart", {
                chart: { type: "column" },
                title: { text: "Cereal Production by Country" },
                xAxis: { type: "category" },
                yAxis: { title: { text: "Production" } },
                series: [{
                    name: "Production",
                    data: chartData
                }]
            });

        } catch (err) {
            console.error(err);
        }
    });
</script>

<main class="container">

    <section class="hero">
        <h1>🌾 Cereal Productions</h1>
        <p>Integración de datos agrícolas globales (proxy + REST API)</p>
        <a href="/integrations">← Volver a integraciones</a>
    </section>

    <section class="section">

        <!-- GRÁFICA -->
        <div class="card">
            <h2>📊 Gráfica de producción</h2>
            <div id="cereal-chart" style="height: 400px;"></div>
        </div>

        <!-- TABLA -->
        <div class="card">
            <h2>📋 Datos completos</h2>

            <div class="table-wrapper">
                <table>
                    <thead>
                        <tr>
                            <th>Country</th>
                            <th>Year</th>
                            <th>Production</th>
                            <th>Yield</th>
                            <th>Population</th>
                        </tr>
                    </thead>

                    <tbody>
                        {#each data as item}
                            <tr>
                                <td>{item.country}</td>
                                <td>{item.year}</td>
                                <td>{item.cereal_production}</td>
                                <td>{item.cereal_yield}</td>
                                <td>{item.population}</td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            </div>
        </div>

    </section>

</main>

<style>
    .container {
        max-width: 1100px;
        margin: auto;
        padding: 2rem;
        font-family: system-ui, sans-serif;
        background: #f8fafc;
        color: #0f172a;
    }

    .hero {
        text-align: center;
        padding: 2rem;
        background: linear-gradient(135deg, #1e3a8a, #3b82f6);
        color: white;
        border-radius: 12px;
        margin-bottom: 2rem;
    }

    .hero a {
        color: white;
        text-decoration: underline;
    }

    .section {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .card {
        background: white;
        border: 1px solid #e2e8f0;
        border-radius: 12px;
        padding: 1rem;
    }

    table {
        width: 100%;
        border-collapse: collapse;
    }

    th, td {
        border-bottom: 1px solid #e2e8f0;
        padding: 8px;
        text-align: left;
    }

    th {
        color: #2563eb;
    }

    .table-wrapper {
        overflow-x: auto;
    }
</style>