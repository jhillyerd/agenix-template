# agenix-template

A NixOS module for templating agenix secrets.  Used with services that do not
support loading secrets directly, you can instead inject secrets into a
configuration or environment file.

# Usage

## Importing

Add the inputs to your `flake.nix`:

```nix
inputs = {
    agenix.url = "github:ryantm/agenix/0.15.0";
    agenix.inputs = {
      nixpkgs.follows = "nixpkgs";
    };

    agenix-template.url = "github:jhillyerd/agenix-template/1.0.0";
};

ouputs = { agenix, agenix-template }: {
        nixosConfigurations = ...
};
```

Then include the modules to your NixOS configuration:

```nix
modules = [
    agenix.nixosModules.default
    agenix-template.nixosModules.default
];
```

**Note:** While the flake is called `agenix-template`, it is referenced in
nix as `age-template` to match the `age` module naming scheme.

## Templating

### Complete example

Setup agenix secrets as normal:

```nix
age.secrets = {
    influxdb-telegraf.file = ./secrets/influxdb-telegraf.age;
};
```

Define the template, mapping secrets into variables:

```nix
# Create an environment file containing the influxdb password.
age-template.files."telegraf-influx.env" = {
    vars.password = config.age.secrets.influxdb-telegraf.path;
    content = ''PASSWORD=$password'';
};
```

Include the template in telegraf options:

```nix
services.telegraf = {
    enable = true;
    extraConfig = {
        outputs.influxdb = with cfg.influxdb; {
            inherit urls database;
            username = user;
            password = "$PASSWORD"; # Grab password from ENV.
        };
        environmentFiles =
            [ config.age-template.files."telegraf-influx.env".path ];
    };
};
```

### Multiple variable example

Multiple secrets within a multi-line template:

```nix
age-template.files = {
    "nomad-secrets.hcl" = {
      vars = {
        consulToken = config.age.secrets.nomad-consul-token.path;
        encrypt = config.age.secrets.nomad-encrypt.path;
      };
      content = ''
        consul {
          token = "$consulToken"
        }
        server {
          encrypt = "$encrypt"
        }
      '';
    };
};
```
