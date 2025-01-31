# Virtuoso Docker Compose

This is a small setup to run a Virtuoso instance in a Docker container and load a dataset into it.  
It is intended to be useful for both long-term and ad-hoc deployments of pre-existing datasets.  
If you want to use Virtuoso to compose a dataset, you can still use this setup as a starting point (there is nothing stopping you from modifying the loaded data).

## Quick start
```bash
# Clone the repository
git clone https://github.com/mjanez/virtuoso-docker.git
cd virtuoso-docker

# Option 1: Generate .env with random passwords
./gen-env.sh

# Option 2: Manually create .env
copy env.example .env
vi .env    # Opens .env to edit
```

Place dataset files into data directory.  
Optionally edit `.env` to set RAM and port settings.
```bash
docker-compose up -d
```

>[!TIP]
> If you're using Windows PowerShell, use `Copy-Item` instead of `copy`:
> ```powershell
> Copy-Item env.example .env
> ```

Once running, by default, you can access:
- SPARQL endpoint at: http://localhost:8890/sparql
- Virtuoso Conductor at: http://localhost:8890/conductor

## Usage

### Database location
The database will be created in subdirectory named database.  
Since this might grow large, you should clone this repository onto a drive with sufficient capacity.

### Initial Setup
Use the `gen-env.sh` script to generate a `.env` file with secure passwords. This script will copy `env.example` and replace the password placeholders with generated passwords. 

You can then customize the .env file to:
- Set memory settings based on your available RAM
- Configure a custom default SPARQL query
- Modify the port settings if needed

>[!NOTE]
> Be aware that the settings in the `.env` file only get applied during the first start of the container. This is a limitation of the Virtuoso container.

### SPARQL Queries
Check the [`SPARQL.md`](SPARQL.md) file for a collection of useful SPARQL queries to explore and analyze DCAT catalogs. These queries are tested and optimized for:
- datos.gob.es SPARQL endpoint: https://datos.gob.es/es/sparql
- European Data Portal SPARQL endpoint: https://data.europa.eu/data/sparql

### Preparing the catalog
Place the catalogs/datasets you want loaded into the data directory.  
Supported file formats:
- `.nt`, `.ttl`, `.nq`, `.owl`, `.rdf`, `.trig`, `.xml`
- Compressed variants: `.gz` and `.bz2`

>[!TIP]
> These are just glob matches in `load.sql`. If you need a different format, you can add it there or open an issue/merge request.

### Managing the container
Use standard Docker Compose commands to manage the container:
```bash
# Start container
docker-compose up -d

# Stop container
docker-compose down

# Update container image
docker-compose up -d --pull
```

### Resetting the database
To reset the database:
```bash
# Stop container and remove database
docker-compose down
rm -rf ./database
docker-compose up -d
```

## License
This work is derived from the original [Virtuoso Docker Compose](https://github.com/dice-group/virtuoso-docker-compose/blob/main/LICENSE) by DICE Group (Copyright 2023), licensed under Apache License 2.0.

This derivative work is licensed under [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

You must give appropriate credit to DICE Group, provide a link to the license, and indicate if changes were made. You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.

![CC BY 4.0](https://i.creativecommons.org/l/by/4.0/88x31.png)