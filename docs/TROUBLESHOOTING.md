# TROUBLESHOOTING

## Pods are not able to mount PVC (especially after longhorn update)

Can be due to issues with multipath. Follow this guide
[https://longhorn.io/kb/troubleshooting-volume-with-multipath/](https://longhorn.io/kb/troubleshooting-volume-with-multipath/)

## Issues with websockets (eg. cannot upload documents to paperless)

Check pfSense/opnSense firewall settings and set it to conservative: [see here](https://github.com/metallb/metallb/issues/654)
