FROM registry.suse.com/bci/bci-base:latest

COPY resolv.conf /etc/resolv.conf
RUN zypper install -y iproute2 iputils

# iperf3 is not delivered by BCI repos -> Manual install (https://iperf.fr/iperf-download.php#manual)
RUN curl -o /usr/lib64/libiperf.so.0 https://iperf.fr/download/ubuntu/libiperf.so.0_3.1.3
RUN curl -o /usr/bin/iperf3 https://iperf.fr/download/ubuntu/iperf3_3.1.3 && chmod +x /usr/bin/iperf3