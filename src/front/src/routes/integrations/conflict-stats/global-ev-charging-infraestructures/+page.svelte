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

        const resEV = await fetch('https://sos2526-16-production.up.railway.app/api/v1/global-ev-charging-infrastructures');
        const resConflictos = await fetch('/api/v2/conflict-stats');

        const evData = await resEV.json();
        const conflictos = await resConflictos.json();

        const maxPower = Math.max(...evData.map(d => d.total_power_kw));

        const evBubbles = evData.map(d => ({
            x: Number(d.year),
            y: d.charging_point,
            r: Math.max(4, Math.round((d.total_power_kw / maxPower) * 28))
        }));

        const conflictBubbles = conflictos.map(d => ({
            x: Number(d.year),
            y: d.intensity_level * 5000,
            r: Math.max(5, d.intensity_level * 8)
        }));

        new window.Chart(chartContainer, {
            type: 'bubble',
            data: {
                datasets: [
                    {
                        label: 'Infraestructura EV',
                        data: evBubbles,
                        backgroundColor: 'rgba(55,138,221,0.45)',
                        borderColor: '#378ADD',
                        borderWidth: 1.5
                    },
                    {
                        label: 'Conflictos armados',
                        data: conflictBubbles,
                        backgroundColor: 'rgba(216,90,48,0.0)',
                        borderColor: '#D85A30',
                        borderWidth: 2
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    tooltip: {
                        callbacks: {
                            label: (ctx) => {
                                if (ctx.datasetIndex === 0) {
                                    const src = evData[ctx.dataIndex];
                                    return `${src.country} (${src.year}): ${src.charging_point.toLocaleString()} puntos`;
                                } else {
                                    const src = conflictos[ctx.dataIndex];
                                    return `${src.location} (${src.year}): intensidad ${src.intensity_level}`;
                                }
                            }
                        }
                    }
                },
                scales: {
                    x: { title: { display: true, text: 'Año' } },
                    y: {
                        min: 0,
                        title: { display: true, text: 'Puntos de carga / Intensidad' },
                        ticks: { callback: v => v >= 1000 ? Math.round(v / 1000) + 'k' : v }
                    }
                }
            }
        });
    });
</script>

<svelte:head>
    <title>Integración - EV Charging vs Conflictos</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2>Infraestructura de Carga EV vs Conflictos Armados</h2>
    <div style="position: relative; height: 420px; width: 100%;">
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>