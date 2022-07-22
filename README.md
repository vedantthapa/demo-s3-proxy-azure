# Azure Blob Storage with S3Proxy Example

Minimal example to run [s3-proxy](https://github.com/gaul/s3proxy) backed by azure blob storage, using [azurite](https://github.com/Azure/Azurite) to emulate Azure blob storage locally.

# Setup

Run `task k3d:create` to create a local k3d kubernetes cluster, and `task k8s:install` to install the kubernetes manifests.

Run `task k8s:delete` to teardown the example and `task k3d:destroy` to delete the k3d cluster.

# Azurite Configuration

Since we are mocking Azure blob storage with Azurite, the URL against which Azure blob storage API calls are made is the following: `http://storage-azurite-service:10000/devstoreaccount1` where `storage-azurite-service` is the name of the kubernetes service assigned to the azurite pods, and `devstoreaccount1` is the default storage account that gets created by Azurite.

## Verify Azurite is up and running with azure CLI

Open a shell into the `az-cli` pod and run the following to create an Azure storage container in the default storage account (using fake default credentials for illustrative purposes).

```bash
az storage container create -n test --connection-string "DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=changeme;BlobEndpoint=http://storage-azurite-service:10000/devstoreaccount1;QueueEndpoint=http://storage-azurite-service:10001/devstoreaccount1;"

```

Verify that the creation was successful by running

```bash
az storage container list --connection-string "DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=changeme;BlobEndpoint=http://storage-azurite-service:10000/devstoreaccount1;QueueEndpoint=http://storage-azurite-service:10001/devstoreaccount1;"
```

# S3Proxy Configuration

See the [following instructions](https://github.com/gaul/s3proxy/issues/262#issuecomment-502251006) on how to configure the s3proxy environment variables to connect to an Azure storage backend.

## Test S3Proxy

Open a shell into the `mc-cli` pod (this pod has the [minio client](https://github.com/minio/mc) installed into the main container, which can be used to make s3 API calls).

Create an alias for the s3proxy instance with the following: `mc alias set s3proxy http://s3proxy:8080 changeme changeme` 

Create a new bucket called `tmp` with `mc mb s3proxy/tmp`

Upload a file to the bucket: `echo test1 > test1.txt && mc cp test1.txt s3proxy/tmp/test1.txt`

Verify that the file was uploaded successfully with `mc cat s3proxy/tmp/test1.txt`

# References

- [Azurite](https://github.com/Azure/Azurite) emulator for Azure storage
- [s3proxy](https://github.com/gaul/s3proxy) to proxy requests against the s3 api to different storage backends