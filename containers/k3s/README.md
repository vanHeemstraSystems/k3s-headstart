# README.md

DEPRECATED:
Instead of using the public k3s-dind (https://github.com/unboundedsystems/k3s-dind), we have to resort to a custom k3s-dind as it will pre-arranged CentOS regarding ip tables and such to allow k3s installation to complete successfully. Current public k3s-didn fails when trying to install on CentOS 7.

NEW:
Instead of using Docker-in-Docker (as it is ill advised to do so), we instead will be having k3s on its own (not dind), based on a CentOS 7 container base image.
