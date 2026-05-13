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

        await fetch('https://sos2526-25.onrender.com/api/v1/average-annual-temperatures/loadInitialData');
        
        const [resTemp, resConflictos] = await Promise.all([
            fetch('https://sos2526-25.onrender.com/api/v1/average-annual-temperatures'),
            fetch('/api/v2/conflict-stats')
        ]);

        const tempData = await resTemp.json();
        const conflictos = await resConflictos.json();

        new window.Chart(chartContainer, {
            type: 'bar',
            data: {
                datasets: [
                    {
                        label: 'Temperatura media anual (°C)',
                        data: tempData.map(d => ({ x: `${d.country} (${d.year})`, y: d.temperature })),
                        backgroundColor: 'rgba(216,90,48,0.55)',
                        borderColor: '#D85A30',
                        borderWidth: 1,
                        yAxisID: 'yTemp'
                    },
                    {
                        label: 'Intensidad de conflicto',
                        data: conflictos.map(d => ({ x: `${d.location} (${d.year})`, y: d.intensity_level })),
                        backgroundColor: 'rgba(55,138,221,0.55)',
                        borderColor: '#378ADD',
                        borderWidth: 1,
                        yAxisID: 'yIntensidad'
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    x: {
                        type: 'category',
                        ticks: { maxRotation: 45, font: { size: 10 } }
                    },
                    yTemp: {
                        type: 'linear',
                        position: 'left',
                        title: { display: true, text: 'Temperatura (°C)' }
                    },
                    yIntensidad: {
                        type: 'linear',
                        position: 'right',
                        min: 0,
                        max: 3,
                        title: { display: true, text: 'Intensidad conflicto' },
                        grid: { drawOnChartArea: false }
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>Integración - Temperatura vs Conflictos</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>Temperatura anual y conflictos armados</h2>
    <div style="position: relative; height: 450px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>