<script>
    import { Line } from 'svelte-chartjs';
    import { data } from './data.js';
    export  let graphData ;
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
<p>{$graphData.length}</p>
<Line {data} options={{ responsive: true }} />
