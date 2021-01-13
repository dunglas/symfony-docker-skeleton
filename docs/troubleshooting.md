# Troubleshooting

## Editing Permissions on Linux

If you work on linux and cannot edit some of the project files right after the first installation, you can run `docker-compose run --rm php chown -R $(id -u):$(id -g) .` to set yourself as owner of the project files that were created by the docker container.

## Fix Chrome/Brave SSL

If you have a TLS trust issues, you can copy the self-signed certificate from Caddy and add it to the trusted certificates :

    # Mac
    $ docker cp $(docker-compose ps -q caddy):/data/caddy/pki/authorities/local/root.crt /tmp/root.crt && sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /tmp/root.crt
    # Linux
    $ docker cp $(docker-compose ps -q caddy):/data/caddy/pki/authorities/local/root.crt /usr/local/share/ca-certificates/root.crt && sudo update-ca-certificates

## HTTPs and Redirects

If Symfony is generating an internal redirect for an `https://` url, but the resulting url is `http://`, you have to uncomment the `TRUSTED_PROXIES` setting in your `.env` file.
For more details see the [Symfony internal redirect documentation](https://symfony.com/doc/current/routing.html#redirecting-to-urls-and-routes-directly-from-a-route).

## Using local TLS certificates

Caddy is configured to generate TLS certificates by using Let's Encrypt, but for local deployments this does not work,
and the internal Caddy certificate should be imported locally (see above).

To use local certificates created with [mkcert](https://github.com/FiloSottile/mkcert) do as follows:

0. Locally install `mkcert`
1. Create the folder storing the certs: `mkdir docker/caddy/certs -p`
2. Generate the certificates for your local host (example: "server-name.localhost"):
   `mkcert -cert-file docker/caddy/certs/tls.pem -key-file docker/caddy/certs/tls.key "server-name.localhost"`
3. Comment out the lines in `./docker-compose.override.yml` file about `CADDY_TLS_CONFIG` environment and volume for the
   `caddy` service
4. Restart your `caddy` container
