# supervisord

<h1>Why Supervisor with Docker ?</h1>
Generally a Docker container starts with a single process when it is launched, for example an Apache daemon or a SSH server daemon. Often though you want to run more than one process in a container. There are a number of ways you can achieve this ranging from using a simple Bash script as the value of your container’s CMD instruction to installing a process management tool.
<h2>Lets start with supervisor to run multiple process in the single container.</h2>
<h2>Create a Dockerfile
Choose Ubuntu LTS 16.04 default image.</h2>
<pre>From ubuntu16.04
Maintainer Arvind Rawat &lt;arvindr226@gmail.com&gt;
</pre>
<h2>Install the apache2, openssh-server and supervisor.</h2>
<pre>RUN sudo apt get update &amp;&amp; apt install -y apache2 openssh-server supervisor
RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd /var/log/supervisor
</pre>
<h2>Configure the ssh for root user in Dockerfile</h2>
<pre>RUN echo 'root:gotechnies' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" &gt;&gt; /etc/profile
</pre>
<h2>Add a supervisor configuration file</h2>
<pre>COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
</pre>
The above configuration will add the supervisord.conf file inthe container while build the docker image.
<h2>Add the below content in the supervisord.conf</h2>
<pre>[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars &amp;&amp; exec /usr/sbin/apache2 -DFOREGROUND"
</pre>
<h2>Exposing ports with running Supervisor </h2>
Let's enable the required default ports to access the http (port 80) and ssh (port 22) in the Dockerfile and write a line to start supervisor execute using the CMD in Dockerfile.
<pre>EXPOSE 22 80
CMD ["/usr/bin/supervisord"]
</pre>
<h2>Your Final complete Dockerfile will look like below.</h2>
<pre>From ubuntu:16.04
Maintainer Arvind Rawat &lt;arvindr226@gmail.com&gt;

RUN apt-get update &amp;&amp; apt-get install -y openssh-server apache2 supervisor
RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd /var/log/supervisor

RUN echo 'root:gotechnies' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" &gt;&gt; /etc/profile
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
EXPOSE 22 80
CMD ["/usr/bin/supervisord"]
</pre>

<h1>Now build your docker image with this Dockerfile</h1>
<pre>
$ docker build -t gotechnies .
</pre>

<h1> Start your docker container like below </h1>
<pre>
 docker run -d -p 80:80 -p 2222:22 gotechnies
</pre>

<h2>Try to connect with ssh server</h2>
<pre>$ ssh root@localhost -p 2222 </pre>

Use the password you have added in the Dockerfile.
<pre>gotechnies</pre>

Check the apache on web browser <code> http://localhost</code>
