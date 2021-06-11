# README.md

Instead of using the public k3s-dind (https://github.com/unboundedsystems/k3s-dind), we have to resort to a custom k3s-dind as it will pre-arranged CentOS regarding ip tables and such to allow k3s installation to complete successfully. Current public k3s-didn fails when trying to install on CentOS 7.
