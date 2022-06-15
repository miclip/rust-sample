# Rust Sample

Basic Rust Web application using the Rocket framework for testing with pack/TBS.

## Pack 

```sh
git clone git@github.com:miclip/rust-sample.git
cd ./rust-sample
pack build rust-sample --buildpack paketo-buildpacks/syft@1.11.3 \
  --buildpack paketo-community/rustup  --buildpack paketo-community/cargo \
  -B paketobuildpacks/builder:tiny  -v
docker run -p 8000:8000  rust-sample
curl localhost:8000/hello/michael/19

Hello, 19 year old named michael!
```

## Tanzu Build Service

First add the buildpacks to the `ClusterStore` 

```sh
kp clusterstore add default --buildpackage paketocommunity/rustup:1.3.0
kp clusterstore add default --buildpackage paketocommunity/cargo:0.5.0
kp clusterstore add default --buildpackage gcr.io/paketo-buildpacks/syft:1.11.3
```

TBS will move the images to the default store (container registry). 

Next create a `Builder` or `ClusterBuilder`:

```sh
kp builder save rust-builder -o ./order.yaml -n builds -s tiny \
  --store default --tag gcr.io/my-project/custom-builders/rust-builder
```

order.yml: 

```yaml
- group: 
  - id: paketo-buildpacks/syft
  - id: paketo-community/rustup
  - id: paketo-community/cargo
  - id: paketo-buildpacks/procfile
    optional: true
```

Finally, create an `Image` resource. 

```sh
 kp image save rust-sample -b rust-builder --git https://github.com/miclip/rust-sample \
   --tag gcr.io/my-project/tbs-examples/rust-sample --namespace builds \
   --git-revision master --env BP_CARGO_EXCLUDE_FOLDERS=Rocket.toml --wait
```

The environment `BP_CARGO_EXCLUDE_FOLDERS=Rocket.toml` is required so `kpack` keeps the `Rocket.toml` configuration file in the container otherwise the Rocket framework won't bind to the correct IP/Port. 

