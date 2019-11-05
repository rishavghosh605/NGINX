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

