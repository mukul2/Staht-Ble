<script>
    import Chart from 'chart.js/auto';

    import { onMount } from 'svelte';
    let prices = {}
    onMount(()  => {
        const ctx = document.getElementById('chart');
        const chart = new Chart(ctx, {
            //Type of the chart
            type: 'line',
            data: {
                //labels on x-axis
                labels: [],
                datasets: [{
                    //The label for the dataset which appears in the legend and tooltips.
                    label: 'Price',
                    //data for the line
                    data: [],
                    //styling of the chart
                    backgroundColor: [
                        'rgba(255, 99, 132, 0.2)',
                    ],
                    borderColor: [
                        'rgba(255, 99, 132, 1)',
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                scales: {
                    //make sure Y-axis starts at 0
                    y: {
                        beginAtZero: true
                    }
                },
            }
        });

        const socket = new WebSocket('wss://ws.coincap.io/prices?assets=bitcoin,ethereum,litecoin,maker,dogecoin,stellar,algorand,basic-attention-coin,compound,celo')
        socket.onmessage = e => {
            let data = JSON.parse(e.data);
            Object.keys(data).forEach(ticker => {

                if (!prices[ticker]) {
                    prices[ticker] = { price: parseFloat(data[ticker])};

                    return;
                }
                prices[ticker] = { price: parseFloat(data[ticker])};
            });
            setTimeout(() => {
                chart.data.labels = Object.keys(prices);
                chart.data.datasets[0].data = Object.values(prices).map(p => p.price);
                chart.update();
            }, 2000);
        };

    });

</script>
<div>
    <canvas id="chart"></canvas>
</div>