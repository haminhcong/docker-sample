FROM ubuntu:16.04

ENV READ_FILE=test.txt

COPY $READ_FILE /root/
COPY print_file.sh /root/

RUN cat /root/print_file.sh

RUN ["chmod", "+x", "/root/print_file.sh"]
WORKDIR /root

ENTRYPOINT ["/bin/sh", "/root/print_file.sh"]
CMD ["-v", "-n5"]