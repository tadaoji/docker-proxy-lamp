FROM mysql:5

ARG DOCKER_LOCATION
ARG MYSQL_ROOT_PASSWORD
RUN ln -sf /usr/share/zoneinfo/${DOCKER_LOCATION:-UTC} /etc/localtime
RUN echo "[client]\npassword=${MYSQL_ROOT_PASSWORD}" > /root/.my.cnf \
&& echo "[mysqld]\n\
character-set-server=utf8mb4\n\
collation-server=utf8mb4_general_ci\n\
explicit-defaults-for-timestamp=1\n\
log-error=/var/log/mysql/error.log\n\
# general_log=1\n\  # logファイルが大きくなるので注意
# general_log_file=/var/log/mysql/query.log\n\  # logファイルが大きくなるので注意
default-storage-engine=InnoDB\n\
" >> /etc/mysql/my.cnf

RUN apt-get update \
&& apt-get install -y \
unzip \
vim
