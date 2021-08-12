# Mosquitto CMD
This file it's collection of command to test Happy GardenPI

## Send message
$ mosquitto_pub -h localhost -m "ciao questo test" -t /HappyGardenPI/{board serial} -u hgardenpi -P hgardenpi -p 1883 -i testclient -d