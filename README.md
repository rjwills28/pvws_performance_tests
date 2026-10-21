# PVWS Performance Tests

Here we provide scripts to test the throughput of array data in PVWS.

## Prerequisites
We assume that EPICS has been installed and that the user can use `softIoc`.

### Install and run PVWS
Follow instructions to install and run PVWS: https://github.com/ornl-epics/pvws.

Ensure that you run PVWS with the following environment variables set (or set in `pvws/docker/setenv.sh` if running PVWS in docker):
```
export PV_THROTTLE_MS=10
export PV_ARRAY_THROTTLE_MS=10
export EPICS_CA_MAX_ARRAY_BYTES=10000000
```

### Set up python virtual environment
```
python -m venv venv
source venv/bin/activate
(venv) pip install -U pip
(venv) pip install numpy aioca websockets
```

## Run the tests
### Start the EPICS DB
We have a simple database file for a waveform record named `test:waveform`.

In one terminal, run the DB it with EPICS `softIoc`
```
softIoc -d throughput.db
```

### Update the waveform PV at 10Hz
In another terminal (with the Python `venv` activated), use the `update_waveform_pv.py` to update the 
`test:waveform` PV at **10Hz** passing in the number of array elements to update:
```
(venv) python update_waveform_pv.py -n 100
```
In the above example we will update `100` array elements.

### Start the client
The client uses a websocket to connect to the running PVWS server on `ws://localhost:8080/pvws/pv`. It then subscribes to the 
`test:waveform` PV. The client waits until it has received 100 updates from PVWS.

We are updating the PV at 10Hz therefore to should take 10 seconds to receive 100 update if PVWS is keeping up. If PVWS drops/skips/falls behind
then it will take longer to receive 100 updates so this is how we test if PVWS is keeping up.

Run the client in another terminal (with the python `venv` activated):
```
(venv) python measure_throughput.py
```
Wait for the script to finish and print out the time and frequency.

## Sample results
To test updating different numbers of array elements:
* Restart the `update_waveform_pv.py` with a different number of sample (`-n XX`)
* Re-run the `measure_throughput.py` client to measure the throughput

| Array size | Rate (Hz) |
|------------|-----------|
| 100        | 10        |
| 1000       | 10        |
| 10000      | 10        |
| 100000     | 10        |
| 1000000    | 1.8       | 
