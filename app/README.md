Reverse_proxy:

1> sudo apt-get nginx
2> sudo service nginx start
3> cd /etc/nginx
Note: sites-enabled is the folder is where the configuration files are put into use
and think of sites available as a store or different config files for different situations
4> cd /sites-available
5> sudo vim node-app
Let node_app be the name of the config file you are designing
Copy paste the code below:

server{

        listen 80;
        location /{
            proxy_pass "http://host_ip:port_number_of_your_application";
        }
}
6> Now we create a soft link for this file in sites-enabled by using:
    sudo ln -s /etc/nginx/sites-available/node-app /etc/nginx/sites-enabled/node-app

7> sudo rm /etc/nginx/sites-available/default
8> sudo rm /etc/nginx/sites-enabled/default
9> sudo /etc/init.d/nginx reload
    or
    sudo service nginx reload
P.S If it does not work thus reinstall nginx
and if you get this bug on going to: sudo systemctl nginx status
go to:https://bugs.launchpad.net/ubuntu/+source/nginx/+bug/1581864
for the workaround.

Load_balancing:

1> We redirect all requests to http on port 80 but websockets are redirected to port 8080

Note:
i)  If you do not want to write the host ip every time you create a new server upstream is the best way to go as by writing the localhost ip as 127.0.0.1 then if we use ec2 instances without elastic ips then on stopping and starting the instance we can still use the nginx config file even though the ip of the instance would have changed.
ii) proxy_http_version is kept as 1.1 as it allows keep-alive connections
upstream nodeapp{
    server 127.0.0.1:3000;
}

server{
    listen 80;
    listen 8080;
    location / {
        proxy_pass http://nodeapp;
        proxy_http_version 1.1;
    }
}

2> Enabling file caching
server{
    location ~* \.(css|js|jpg|png|jpeg|gif)${
        expires 200h;
    }
}



#Load Balancing

Load balancing with nginx uses a round-robin algorithm by default if no other method is defined, like in the first example above. With round-robin scheme each server is selected in turns according to the order you set them in the load-balancer.conf file. This balances the number of requests equally for short operations.

Least connections based load balancing is another straightforward method. As the name suggests, this method directs the requests to the server with the least active connections at that time. It works more fairly than round-robin would with applications where requests might sometimes take longer to complete.

To enable least connections balancing method, add the parameter least_conn to your upstream section as shown in the example below.

IP hashing uses the visitors IP address as a key to determine which host should be selected to service the request. This allows the visitors to be each time directed to the same server, granted that the server is available and the visitor’s IP address hasn’t changed.

To use this method, add the ip_hash -parameter to your upstream segment like in the example underneath.

To load balance with different servers we use
upstream app_name{
    least_conn;
    or
    ip_hash;//We can add"ip_hash" when we need to use the same server a particular client       //has connected to, basically when using sockets or the applcation is stateless
       // This strategy is called sticky sessions
    list of servers
    server2 weight=1
    server1 weight=n//the weight parameter instructs NGNX to pass n times as many connections to server1 as compared to server2
}


#Health checks
In order to know which servers are available, nginx’s implementations of reverse proxy includes passive server health checks. If a server fails to respond to a request or replies with an error, nginx will note the server has failed. It will try to avoid forwarding connections to that server for a time.

The number of consecutive unsuccessful connection attempts within a certain time period can be defined in the load balancer configuration file. Set a parameter max_fails to the server lines. By default, when no max_fails is specified, this value is set to 1. Optionally setting the max_fails to 0 will disable health checks to that server.

If max_fails is set to a value greater than 1 the subsequent fails must happen within a specific time frame for the fails to count. This time frame is specified by a parameter fail_timeout, which also defines how long the server should be considered failed. By default, the fail_timeout is set to 10 seconds.

After a server is marked failed and the time set by fail_timeout has passed, nginx will begin to gracefully probe the server with client requests. If the probes return successful, the server is again marked live and included in the load balancing as normal.

upstream backend {
   server 10.1.0.101 weight=5;
   server 10.1.0.102 max_fails=3 fail_timeout=30s;
   server 10.1.0.103;
}




upstream app{
        ip_hash;
        server localhost:8080;
        server localhost:8081;
}
server{
        listen 80;
        location / {
                proxy_pass "http://app/";
        }

}
