# Assets

When deploying with Kamal, it will first start the new version of the application before stopping the old version. That means that there is a short period of time where both versions are running. That's why it's possible that the old version gets a request meant for the new version and vice versa, resulting in 404s.

For assets, Kamal offers a feature that'll avoid getting such 404s while deploying. It basically copies new assets to the old version of the application and it copies old assets to the new version of the application. This works by using the `asset_path` config option, like:

```yml
asset_path: /rails/public/assets
```

The deploy process looks like this:

### 1. [Extract Assets](https://github.com/basecamp/kamal/blob/v1.1.0/lib/kamal/commands/app/assets.rb#L2-L12)

Before starting the new version of the application, run a container that sleeps and copy the assets out of that into a directory on the host:

```
$ docker run --name foo-web-production-assets --detach --rm <image>:latest sleep 1000000
$ docker cp -L foo-web-production-assets:/rails/public/assets/. .kamal/assets/extracted/foo-web-production-<new-version>
$ docker stop -t 1 foo-web-production-assets
```

### 2. [Sync Asset Volumes](https://github.com/basecamp/kamal/blob/v1.1.0/lib/kamal/commands/app/assets.rb#L14-L28)

After extracting assets, they will be copied into a volume directory on the host:

```
$ mkdir -p .kamal/assets/volumes/foo-web-production-<new-version>
$ cp -rnT .kamal/assets/extracted/foo-web-production-<new-version> .kamal/assets/volumes/foo-web-production-<new-version>
```

… and, if there's an old version of the application running on the host, add the new assets to the old version of the application and the old assets to the new version of the application:

```
$ cp -rnT .kamal/assets/extracted/foo-web-production-<new-version> .kamal/assets/volumes/foo-web-production-<old-version> || true
$ cp -rnT .kamal/assets/extracted/foo-web-production-<old-version> .kamal/assets/volumes/foo-web-production-<new-version> || true
```

### 3. Run new Application

The new version of the application will be started and the volume directory will be mounted:

```
$ docker run --detach --name foo-web-production-<new-version> <options> --volume $(pwd)/.kamal/assets/volumes/foo-web-production-<new-version>:/rails/public/assets <image>:<new-version>
```

By doing so, the application's asset directory is kind of discarded and the host's volume directory is used instead.

### 4. Stop old Application

If there's an old version of the application running on the host, stop it:

```
$ docker container ls --all --filter name=^foo-web-production-<old-version>$ --quiet | xargs docker stop
```

### 5. [Clean Up Assets](https://github.com/basecamp/kamal/blob/v1.1.0/lib/kamal/commands/app/assets.rb#L30-L34)

If there was an old version of the application running on the host, clean up any assets from any old version. It only keeps the new extracted assets and the new volume directory:

```
$ find .kamal/assets/extracted -maxdepth 1 -name 'foo-web-production-*' ! -name foo-web-production-<new-version> -exec rm -rf "{}" +
$ find .kamal/assets/volumes -maxdepth 1 -name 'foo-web-production-*' ! -name foo-web-production-<new-version> -exec rm -rf "{}" +
```
