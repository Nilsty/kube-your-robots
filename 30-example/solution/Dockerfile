FROM ppodgorsek/robot-framework:latest

USER root

COPY example-grafana-login.robot /opt/robotframework/tests/

COPY run-tests-in-virtual-screen.sh /opt/robotframework/bin/

RUN pip3 install \
  --no-cache-dir \
  robotframework-kubelibrary==0.8.7
