## Transfer an OpenShift operator to a disconnected environment

1. install oc, opm, podman and grpcurl
2. login to the redhat registry:

   ```
   podman login registry.redhat.io
   ```
3. Create a pod which contains the redhat operator index:
   ```
   podman run -p50051:50051 \
   -it registry.redhat.io/redhat/redhat-operator-index:v<oc-major>
   ```
4. Save into a file all the existing packages in the redhat opeartor index:
    ```
    grpcurl -plaintext localhost:50051 api.Registry/ListPackages > packages.out
    ```
5. Find your required operator:
    ```
    cat packages.out  | grep ocs-operator
    ```
6. Prune the index image to include only your required operator, the target tag should be a registry you would push the new index image to:
    ```
    opm index prune -f registry.redhat.io/redhat/redhat-operator-index:v4.6 -p ocs-operator -t localhost:5000/ocs/ocs-index:v4.6
    ```
7.  If you don't want to push your image to a public registry, you can setup simply a registry of your own on your local host:
    ```
    podman run -d -p 5000:5000 --restart=always --name registry registry:2.7
    ```
    
    You would also need to add it to the insecure registries array in the `/etc/containers/registries.conf` file.
    
8. Push the image:
  ```
  podman push localhost:5000/ocs/ocs-index:v4.6
  ```
  
9. Now you can mirror the needed files for the operator to your filesystem:
  ```
  oc adm catalog mirror --insecure localhost:5000/ocs/ocs-index:v4.6 file:///local/index
  ```
  
10. Copy the generated directories into removable media
