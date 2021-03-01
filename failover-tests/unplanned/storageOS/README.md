# Steps

Deploy elasticsearch

- run rally nyc_taxis

- wait for approx 40min

- shutdown one of the hosts

- wait approx 5min for unready state adn pods to try to terminate

- delete k8s node object

- pods will now be removed and initialise on different node

- log in storageOS UI and delete failed node from there

- after 5 min, storageOS csi will reattach volume to new pod 

- rally should be stil running, wait to complete and grab result

- take screemshot of grafana dashboards es exporter