<script >
    //variable Init
    import {onMount} from "svelte";
    import Chart from "../components/Chart.svelte";
    var isAvailable = false;
    var deviceServer;

    var connectedDevice;

    let array = [1,2,3,4];
    $: gauseReading = 0.0;

    $: dataArray = [];
    //function to be called on click
    onMount(() => {
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
        gauseReading = num.toFixed(1);
        //counter.push(num);

        dataArray.push(num*100);


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
</script>

<!--<main>-->
<!--    <Chart  graphData = {dataArray}/>-->
<!--</main>-->
<button on:click={requestBluetoothDevices} disabled={!isAvailable} class="btn btn-primary">
    Connect Bluetooth Device
</button>


<h1>{gauseReading}</h1>

