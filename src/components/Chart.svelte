<script>
    import { Line } from 'svelte-chartjs';
    import { data } from './data.js';
    import {
        Chart as ChartJS,
        Title,
        Tooltip,
        Legend,
        LineElement,
        LinearScale,
        PointElement,
        CategoryScale,
    } from 'chart.js';

    ChartJS.register(
        Title,
        Tooltip,
        Legend,
        LineElement,
        LinearScale,
        PointElement,
        CategoryScale
    );


</script>
<!--<h1>{$graphData.length}</h1>-->
<Line {data} options={{ responsive: true }} />
