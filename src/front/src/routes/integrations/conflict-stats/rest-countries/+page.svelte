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

        const res = await fetch('https://restcountries.com/v3.1/all?fields=region,population');
        const countries = await res.json();

        // Agrupar población total por región
        const regionMap = {};
        countries.forEach(c => {
            const region = c.region || 'Unknown';
            const pop = c.population || 0;
            regionMap[region] = (regionMap[region] || 0) + pop;
        });

        const labels = Object.keys(regionMap).sort();
        const data = labels.map(r => Math.round(regionMap[r] / 1_000_000));

        new window.Chart(chartContainer, {
            type: 'doughnut',
            data: {
                labels,
                datasets: [{
                    data,
                    backgroundColor: [
                        '#378ADD',
                        '#D85A30',
                        '#2ecc71',
                        '#f39c12',
                        '#9b59b6',
                        '#1abc9c',
                        '#e74c3c'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    tooltip: {
                        callbacks: {
                            label: (ctx) => `${ctx.label}: ${ctx.raw.toLocaleString()} M hab.`
                        }
                    },
                    legend: {
                        position: 'right'
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>REST Countries - Población por región</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>Población mundial por región</h2>
    <div style="position: relative; height: 450px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>