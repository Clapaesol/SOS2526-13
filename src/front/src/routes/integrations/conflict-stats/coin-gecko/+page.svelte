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
            'https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=10&page=1'
        );
        const coins = await res.json();

        const labels = coins.map(c => c.symbol.toUpperCase());
        const marketCaps = coins.map(c => c.market_cap);
        const volumes = coins.map(c => c.total_volume);
        const prices = coins.map(c => c.current_price);

        // Normalizar cada serie a escala 0-100 para que el radar sea legible
        const normalize = arr => {
            const max = Math.max(...arr);
            return arr.map(v => Math.round((v / max) * 100));
        };

        new window.Chart(chartContainer, {
            type: 'radar',
            data: {
                labels,
                datasets: [
                    {
                        label: 'Capitalización (normalizada)',
                        data: normalize(marketCaps),
                        backgroundColor: 'rgba(55,138,221,0.2)',
                        borderColor: '#378ADD',
                        borderWidth: 2,
                        pointRadius: 3
                    },
                    {
                        label: 'Volumen 24h (normalizado)',
                        data: normalize(volumes),
                        backgroundColor: 'rgba(216,90,48,0.2)',
                        borderColor: '#D85A30',
                        borderWidth: 2,
                        pointRadius: 3
                    },
                    {
                        label: 'Precio USD (normalizado)',
                        data: normalize(prices),
                        backgroundColor: 'rgba(46,204,113,0.2)',
                        borderColor: '#2ecc71',
                        borderWidth: 2,
                        pointRadius: 3
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'top' },
                    tooltip: {
                        callbacks: {
                            label: (ctx) => {
                                const i = ctx.dataIndex;
                                const c = coins[i];
                                if (ctx.datasetIndex === 0) return `Cap: $${c.market_cap.toLocaleString()}`;
                                if (ctx.datasetIndex === 1) return `Vol: $${c.total_volume.toLocaleString()}`;
                                return `Precio: $${c.current_price.toLocaleString()}`;
                            }
                        }
                    }
                },
                scales: {
                    r: {
                        min: 0,
                        max: 100,
                        ticks: { stepSize: 25 }
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>CoinGecko - Top 10 Criptomonedas</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>Top 10 criptomonedas — capitalización, volumen y precio (normalizado)</h2>
    <div style="position: relative; height: 500px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>