<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;

    onMount(async () => {
        if (!browser) return;

        await new Promise((resolve, reject) => {
            if (window.Chart) return resolve();
            const script = document.createElement('script');
            script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });

        const res = await fetch(
            'https://api.open-meteo.com/v1/forecast?latitude=37.38&longitude=-5.97&hourly=temperature_2m,precipitation_probability&forecast_days=7&timezone=Europe%2FMadrid'
        );
        const data = await res.json();

        // Una lectura cada 6 horas para no saturar
        const times = data.hourly.time.filter((_, i) => i % 6 === 0);
        const temps = data.hourly.temperature_2m.filter((_, i) => i % 6 === 0);
        const precip = data.hourly.precipitation_probability.filter((_, i) => i % 6 === 0);

        const labels = times.map(t => t.replace('T', ' '));

        new window.Chart(chartContainer, {
            type: 'bar',
            data: {
                labels,
                datasets: [
                    {
                        label: 'Temperatura (°C)',
                        data: temps,
                        backgroundColor: 'rgba(216,90,48,0.7)',
                        borderColor: '#D85A30',
                        borderWidth: 1,
                        yAxisID: 'yTemp'
                    },
                    {
                        label: 'Prob. precipitación (%)',
                        data: precip,
                        backgroundColor: 'rgba(55,138,221,0.7)',
                        borderColor: '#378ADD',
                        borderWidth: 1,
                        yAxisID: 'yPrecip'
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'top' }
                },
                scales: {
                    x: {
                        ticks: { maxRotation: 45, font: { size: 10 } }
                    },
                    yTemp: {
                        type: 'linear',
                        position: 'left',
                        title: { display: true, text: 'Temperatura (°C)' }
                    },
                    yPrecip: {
                        type: 'linear',
                        position: 'right',
                        min: 0,
                        max: 100,
                        title: { display: true, text: 'Precipitación (%)' },
                        grid: { drawOnChartArea: false }
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>Open-Meteo - Previsión Sevilla</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>Previsión meteorológica — Sevilla (7 días)</h2>
    <div style="position: relative; height: 450px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>