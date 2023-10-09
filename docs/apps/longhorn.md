# Longhorn

## Copy data to/from volume

One of the ways to access or move data in or out of a PV, is to create a temporary pod that mounts it, shell into it and do whatever we need. Here's a sample manifest to do it:

```yaml

---
apiVersion: v1
kind: Pod
metadata:
  name: copy-volume-data
  # PV namespace
  namespace: home-cloud
spec:
  containers:
    - name: restore-to-file
      command:
      - sleep
      - "86400"
      image: ubuntu
      imagePullPolicy: IfNotPresent
      securityContext:
        privileged: true
      volumeMounts:
        - name: other
          mountPath: /mnt/other
        - name: longhorn
          mountPath: /mnt/longhorn
  volumes:
    - name: other
      # secondary volume to mount, if needed, eg. an NFS share
      nfs:
        server: truenas.aresu.eu
        path: /mnt/tank/documents
        readOnly: true
    - name: longhorn
      persistentVolumeClaim:
        claimName: paperless-media-v1
  restartPolicy: Never
```

## Restore volume backup to file

Longhorn backups are saved in an opaque format, so to access one we can have longhorn export it in a standard virtual disk format, by running a pod from a manifest like this:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: restore-to-file
  namespace: longhorn-system
spec:
  # specific host to get the output file from
  nodeName: kube-nuc-01
  containers:
    - name: restore-to-file
      command:
        # set backup url and output filename here
        - /bin/sh
        - -c
        - longhorn backup restore-to-file
          'nfs://10.17.1.2:/mnt/tank/longhorn-backups?backup=backup-26eae1744e9f4e04&volume=plex-config-v1'
          --output-file '/tmp/restore/plex-config-v1.qcow2'
          --output-format qcow2
      # version >= v0.4.1
      image: longhornio/longhorn-engine:v1.4.2
      imagePullPolicy: IfNotPresent
      securityContext:
        privileged: true
      volumeMounts:
        - name: host-directory
          mountPath: /tmp/restore
      # set Backup Target Credentials here if needed.
      # env:
      # - name: AWS_ACCESS_KEY_ID
      #   valueFrom:
      #     secretKeyRef:
      #       name: <S3_SECRET_NAME>
      #       key: AWS_ACCESS_KEY_ID
      # - name: AWS_SECRET_ACCESS_KEY
      #   valueFrom:
      #     secretKeyRef:
      #       name: <S3_SECRET_NAME>
      #       key: AWS_SECRET_ACCESS_KEY
      # - name: AWS_ENDPOINTS
      #   valueFrom:
      #     secretKeyRef:
      #       name: <S3_SECRET_NAME>
      #       key: AWS_ENDPOINTS
  volumes:
    - name: host-directory
      hostPath:
        # the output file destination on the host
        path: /tmp/restore
  restartPolicy: Never
```
