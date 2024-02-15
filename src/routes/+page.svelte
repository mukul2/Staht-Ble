<script >
    //variable Init
    import Chart from 'chart.js/auto';
    import { Line } from 'svelte-chartjs';

    import {onMount} from "svelte";
    import LineGraph from "../components/LineGraph.svelte";
    import data2 from "../components/data2.js";
    // import {data} from "../components/data.js";
    let chartData;
    var isAvailable = false;
    var deviceServer;

    var connectedDevice;

    let array = [1,2,3,4];
    $: gauseReading = 0.0;

    $: dataArray = [];
    $: TimeArray = [];

    var  chart;

    //function to be called on click
    onMount(() => {

        const ctx = document.getElementById('chart');

          chart = new Chart(ctx, {
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
                elements: {
                    point:{
                        radius: 0
                    }
                },
                animation: {
                    duration: 0
                },
                scales: {
                    //make sure Y-axis starts at 0
                    y: {
                        beginAtZero: true
                    }
                },
            }
        });


        enableBluetooth();
    });
    async function enableBluetooth() {
        try {
            isAvailable = await navigator.bluetooth.getAvailability();
            console.log('Bluetooth is enabled: ', isAvailable);
            return Promise.resolve();
        } catch (e) {
            console.error(e);
            isAvailable = false;
            return Promise.reject();
        }
    }
    function onAccelerometerChanged(event) {
        const characteristic = event.target;
        //console.log('Accelerometer data changed', characteristic.value)
        var a1 = characteristic.value.getUint8(0);
        var a2 = characteristic.value.getUint8(1);
        var a3 = characteristic.value.getUint8(2);
        var a4 = characteristic.value.getUint8(3);

        array = [a1, a2, a3, a4];
        var buf = new ArrayBuffer(4);
        var view = new DataView(buf);
        array.forEach(function (b, i) {
            view.setUint8(i, b);
        });
        var num = view.getFloat32(0);
        gauseReading = num.toFixed(5);
        //counter.push(num);

        dataArray.push(num.toFixed(1));
        TimeArray.push(new Date().toLocaleTimeString());
        chart.data.labels = TimeArray;
        chart.data.datasets[0].data = dataArray;
        chart.update();

    }
    async function subscribeToAccelerometerCharacteristic() {
        try {
            const accService = await deviceServer.getPrimaryService('f0001110-0451-4000-b000-000000000000');
            const accDataCharacteristic = await accService.getCharacteristic('f0001113-0451-4000-b000-000000000000');
            accDataCharacteristic.oncharacteristicvaluechanged = onAccelerometerChanged;
            accDataCharacteristic.startNotifications();
        } catch (e) {
            console.error(e);
        }
    }
    async function requestBluetoothDevices() {
        try {
            console.log('Requesting Bluetooth Devices');
            const device = await navigator.bluetooth.requestDevice({
                filters: [{ namePrefix: 'Staht' },{ namePrefix: 'BLE' }],
                optionalServices: [
                    'f0001110-0451-4000-b000-000000000000',

                ]
            });
            deviceServer = await device.gatt.connect();
            connectedDevice = deviceServer.device;
            subscribeToAccelerometerCharacteristic();
            return Promise.resolve();
        } catch (e) {
            console.error(e);
            return Promise.reject();
        }
    }

    $: data = {
        labels: TimeArray,
        datasets: [
            {
                label: 'My First dataset',


                data: dataArray,
            }
        ],
    }


</script>
<!--<div>-->
<!--    <canvas id="chart"></canvas>-->
<!--</div>-->
<main>
    <button on:click={requestBluetoothDevices} disabled={!isAvailable} class="btn btn-primary">
        Connect Bluetooth Device
    </button>
    <h1>{gauseReading}</h1>
    <div>
        <canvas id="chart"></canvas>
    </div>
</main>





<!--<h1>{gauseReading}</h1>-->

