# You can install Prometheus in multiple ways as listed below :
    1. Using Helm charts
    2. Using Docker containers
    3. Using official commands listed on Prometheus Documentation.

 # 1. Using Helm Charts :
       1. Add helm repo
            helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      2. Update helm repo
            helm repo update
      3. Install helm
            helm install prometheus prometheus-community/prometheus
      4. Expose Prometheus Service
           This is required to access prometheus-server using your browser.
           kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext

 # 2. Using Docker containers :
      If you have your own prometheus.yml file then use this command
         docker run -d -p 9090:9090 -v ~/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus - 
         config.file=/etc/prometheus/prometheus.yml -storage.local.path=/prometheus -storage.local.memory-chunks=10000
         This command is quite long and contains many command line options. Let’s take a look at it in more detail:
       1. The -d option starts the P rometheus container in detached mode, meaning that the container will be started in the background 
          and will not be terminated by pressing CTRL+C.
       2. The -p 9090:9090 option exposes Prometheus’s web port (9090) and makes it reachable via the external IP address of the host 
          system.
       3. The -v [...] option mounts the prometheus.yml configuration file from the host filesystem into the location within the 
          container where Prometheus expects it (/etc/prometheus/prometheus.yml).
       4. The -config.file option is set accordingly to the location of the Prometheus configuration file within in the container.
       5. The -storage.local.path option configures the metrics storage location within the container.
       6. Finally, the -storage.local.memory-chunks option adjusts Prometheus’s memory usage to the host system’s very small amount of 
          RAM (only 512MB) and small number of stored time series in this tutorial (just under 1000). 
         It instructs Prometheus to keep only 10000 sample chunks in memory (roughly 10 chunks per series), instead of the default of 
         1048576. This is a value you will definitely need to tune when running Prometheus on a machine 
         with more RAM and when storing more time series. Refer to Prometheus’s storage documentation for more details around this.

   If you want to do everything from scratch use 
     docker container run -itd --name prometheus -p 9090:9090 prom/prometheus

 # 3. Using official commands listed on Prometheus Documentation (On Linux) :
        1. wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
        2. tar -xvf ........tar.gz
        3. ./prometheus  -- It will start the application
        4. Create an service so that your prometheus will run continuously.
            sudo cp -r . /usr/local/bin/prometheus
            sudo vi /etc/systemd/system/prometheus.service     # Add data from official website 
            [Unit]
            Description=Prometheus
            After=network.target

            [Service]
            Type=simple
            ExecStart=/usr/local/bin/prometheus/prometheus --config.file /usr/local/bin/prometheus/prometheus.yml 
              
            [Install]
            WantedBy=multi-user.target

        sudo systemctl daemon-reload
        sudo service prometheus start
          sudo service prometheus status
          Now check on browser with http://Public_ip:9090

# Node Exporter :
      Works as matrics exporter.
      Now we can install node exporter which acts as agent.
        1. wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
        2. tar -xvf *.tar.gz
        3. cd node*
        4. cp -r node_exporter /usr/local/bin/
        5. vi /etc/systemd/system/node_exporter.service
           [Unit]
           Description=Node_Exporter
           after=network.target

           [Service]
           type=simple
           ExecStart=/usr/local/bin/node_exporter 

           [Install]
           WantedBy=multi-user.target

        6. Restart the follo servers:
            systemctl daemon-relode
            service node_exporter start
            service node_exporter status
            http://localhost:9100

        7. To check in prometheus we need to add these values in prometheus.yml file under target section as.
               - job_name: 'node-exporter'
                 static_configs:
                  - targets: ['localhost:9100']
        8. Restart the follo servers:
              service prometheus restart
              service prometheus status

# Alert Manager :
    Now will install alert manager which acts as alerting system
        1. wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
        2. tar -xvf *.tar.gz
        3. cp -r . /usr/local/bin/
        4. vi /etc/systemd/system/alert_manager.service
            [Unit]
            Description=Alert_Manager
            after=network.target

            [Service]
            type=simple
            ExecStart=/usr/local/bin/alert_manager --config.file /usr/local/bin/alert_manager/alert_manager.yml

            [Install]
            WantedBy=multi-user.target
       
        5. Restart the follo servers:
             systemctl daemon-relode
             service alert_manager start
             service alert_manager status
             http://localhost:9093

        6. First we have to add rules under prometheus directory by creating prometheus_rules.yml file and adding below content
           groups:
           - name: alert_rules
             rules:
               - alert: InstanceDown
                 expr: up == 0
                 for: 1m
                 labels:
                   severity: critical
                 annotations:
                   summary: "Instance [{{ $labels.instance }}] down"
                   description: "[{{ $labels.instance }}] of job [{{ $labels.job }}] has been down for more than 1 minute."
	   
        7. Save it and test it with the promtool
                ./promtool check rules prometheus_rules.yml   
		
	 
        8. To check in prometheus we need to add these values in prometheus.yml file under alerting section as.
              
        9. Restart the follo servers:
              service prometheus restart
               service prometheus status

      
