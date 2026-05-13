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
            'https://api.worldbank.org/v2/country/us;cn;de;fr;gb;jp;in;br;es;za/indicator/NY.GDP.MKTP.CD?format=json&mrv=1&per_page=10'
        );
        const json = await res.json();

        // Los datos están en json[1]; filtrar nulls y ordenar de mayor a menor
        const records = json[1]
            .filter(d => d.value !== null)
            .sort((a, b) => b.value - a.value);

        const labels = records.map(d => d.country.value);
        const values = records.map(d => Math.round(d.value / 1e9)); // en miles de millones USD

        new window.Chart(chartContainer, {
            type: 'polarArea',
            data: {
                labels,
                datasets: [{
                    data: values,
                    backgroundColor: [
                        'rgba(55,138,221,0.6)',
                        'rgba(216,90,48,0.6)',
                        'rgba(46,204,113,0.6)',
                        'rgba(243,156,18,0.6)',
                        'rgba(155,89,182,0.6)',
                        'rgba(26,188,156,0.6)',
                        'rgba(231,76,60,0.6)',
                        'rgba(52,152,219,0.6)',
                        'rgba(230,126,34,0.6)',
                        'rgba(149,165,166,0.6)'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'right' },
                    tooltip: {
                        callbacks: {
                            label: (ctx) => `$${ctx.raw.toLocaleString()} B USD`
                        }
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>World Bank - PIB por país</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>PIB por país — World Bank (valor más reciente)</h2>
    <div style="position: relative; height: 500px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>