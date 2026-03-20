# 🥪 The Jaffle Shop (for ClickHouse) 🦘

This is a sandbox project for exploring the basic functionality and latest features of dbt with Clickhouse. It's based on the original dbt [jaffle shop project](https://github.com/dbt-labs/jaffle-shop).

We made specific changes to make it work with Clickhouse. We also introduced specific documentation to help you kickstart your ClickHouse project.

You may still find some incompatibilities with Clickhouse. If you do, please open an issue in the GitHub repository. Also feel free to contribute:
- Examples of interesting dbt resources that interact in an special way with ClickHouse.
- Any good practice that you think should be explained.
- Examples of workarounds needed to implement certain features.
- Features listed in the `jaffle project` that may not be compatible with the current `dbt-clickhouse` adapter like references to dbt cloud.


<div>
 <a href="https://www.loom.com/share/a90b383eea594a0ea41e91af394b2811?t=0&sid=da832f06-c08e-43e7-acae-a2a3d8d191bd">
   <p>Welcome to the Jaffle Shop - Watch Intro Video</p>
 </a>
 <a href="https://www.loom.com/share/a90b383eea594a0ea41e91af394b2811?t=0&sid=da832f06-c08e-43e7-acae-a2a3d8d191bd">
   <img style="max-width:300px;" src="https://cdn.loom.com/sessions/thumbnails/a90b383eea594a0ea41e91af394b2811-with-play.gif">
 </a>
</div>

## Table of contents

1. [Prerequisites](#-prerequisites)
2. [Create new repo from template](#-create-new-repo-from-template)
3. [Platform setup](#%EF%B8%8F-platform-setup)
   1. [Load the data](#-load-the-data)
4. [Project setup](#%EF%B8%8F-project-setup)
5. [Going further](#-going-further)
   1. [Working with a larger dataset](#-working-with-a-larger-dataset)
      1. [Load the data from S3](#-load-the-data-from-s3)
      2. [Generate via `jafgen` and seed the data with dbt Core](#-generate-via-jafgen-and-seed-the-data-with-dbt-core)
   2. [Pre-commit and SQLFluff](#-pre-commit-and-sqlfluff)

## 💾 Prerequisites

- Python version between 3.9 to 3.12 (>=3.11 recommended)
- Access to a ClickHouse cluster. For example:
    - A cloud cluster (using ClickHouse Cloud)
    - A local cluster (using docker)

## 🏗️ Platform setup

### Installing dbt

Create a local Python environment and activate it:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install dbt and dbt-clickhouse locally:

```bash
pip install dbt-core dbt-clickhouse
```

Install dbt-dependencies
```bash
dbt deps
```

### Configuring a ClickHouse cluster

#### Local cluster (using docker)

Start a local ClickHouse cluster locally by using docker:

```bash
./clickhouse-docker/start_ch_cluster.sh
```

Rename `profiles-local.yml` to `profiles.yml`. It should work without more changes.

And test dbt can connect to ch:

```bash
dbt debug
```

You can later stop the cluster by using:

```bash
./clickhouse-docker/stop_ch_cluster.sh
```

#### Cloud cluster (using ClickHouse Cloud)

Rename `profiles-cloud.yml` to `profiles.yml`.It should work without more changes.

Set the environment variables:

```bash
export DBT_HOST="my_cluster.my_region.clickhouse.com"
export DBT_USER="default"
export DBT_PASSWORD="my_password"
```

And test dbt can connect to ch:

```bash
dbt debug
```

### 📊 Load the data

There are a few ways to load the data for the project:

- **Using the sample data in the repo**. Seeds are static data files in CSV format that dbt will upload, usually for reference models, like US zip codes mapped to country regions for example, but in this case the feature is hacked to do some data ingestion. This is not what seeds are meant to be used for (dbt is not a data loading tool), but it's useful for this project to give you some data to get going with quickly. Run the command below and when it's done either delete the `seeds/jaffle-data` folder, remove `jaffle-data` config from the `dbt_project.yml`, or ideally, both.

```bash
dbt seed --full-refresh --vars '{"load_source_data": true}'
```

- **Load the data via S3**. If you'd prefer a larger dataset (6 years instead of 1), you can also copy the data from a public S3 bucket to your warehouse into a schema called `raw` in your `jaffle_shop` database. [This is discussed here](#-load-the-data-from-s3).

- **Generate a larger dataset on the command line**. If you're comfortable with command line basics, you can generate as many years of data as you'd like (up to 10) to load into your warehouse. [This is discussed here](#-generate-via-jafgen-and-seed-the-data-with-dbt-core).

## 👷🏻‍♀️ Project setup

Once your development platform of choice and dependencies are set up, use the following steps to get the project ready for whatever you'd like to do with it.

1. Run a `dbt build` to build the project.

### 🏁 Checkpoint

The following should now be done:

- Synthetic data loaded into your warehouse
- Development environment set up and ready to go
- The project built and tested

You're free to explore the Jaffle Shop from here, or if you want to learn more about [generating a larger dataset](#-working-with-a-larger-dataset), or [setting up pre-commit hooks](#-pre-commit-and-sqlfluff) to standardize formatting and linting workflows, carry on!

## 🌅 Going further

> [!NOTE]
> 🐉 Here be dragons! The following sections are for folks who are comfortable with the basics and want to explore more advanced topics. If you're just getting started, it's okay to skip these for now and come back later.

### 🏭 Working with a larger dataset

There are two ways to work with a larger dataset than the default one year of data that comes with the project:

1. **Load the data from S3** which will let you access the canonical 6 year dataset the project is tested against.

2. **Generate via [`jafgen`](https://github.com/dbt-labs/jaffle-shop-generator) and seed the data with dbt Core** which will allow you to generate up to 10 years of data.

#### 💾 Load the data from S3

To load the data from S3, consult the [dbt Documentation's Quickstart Guides](https://docs.getdbt.com/guides) for your data platform to see how to copy data from an S3 bucket to your warehouse. The S3 bucket URIs of the tables you want to copy into your `raw` schema are:

| table name        | S3 URI                                                           | Direct Download Link                                                                                     | Schema                                                                                                    |
|-------------------|------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `raw_customers` | `s3://dbt-tutorial-public/long_term_dataset/raw_customers.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_customers.csv) | `(id text, name text)` |
| `raw_orders` | `s3://dbt-tutorial-public/long_term_dataset/raw_orders.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_orders.csv) | `(id text, customer text, ordered_at datetime, store_id text, subtotal int, tax_paid int, order_total int)` |
| `raw_order_items` | `s3://dbt-tutorial-public/long_term_dataset/raw_order_items.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_order_items.csv) | `(id text, order_id text, sku text)` |
| `raw_products` | `s3://dbt-tutorial-public/long_term_dataset/raw_products.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_products.csv) | `(sku text, name text, type text, price int, description text)` |
| `raw_supplies` | `s3://dbt-tutorial-public/long_term_dataset/raw_supplies.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_supplies.csv) | `(id text, name text, cost int, perishable boolean, sku text)` |
| `raw_stores` | `s3://dbt-tutorial-public/long_term_dataset/raw_stores.csv` | [Download](https://dbt-tutorial-public.s3.us-west-2.amazonaws.com/long_term_dataset/raw_stores.csv) | `(id text, name text, opened_at datetime, tax_rate float)` |

#### 🌱 Generate via `jafgen` and seed the data with dbt Core

You'll need to be working on the command line for this option. If you're more comfortable working via web apps, the above method is the path you'll need. [`jafgen`](https://github.com/dbt-labs/jaffle-shop-generator) is a simple tool for generating synthetic Jaffle Shop data that is maintained on a volunteer-basis by dbt Labs employees. This project is more interesting with a larger dataset generated and uploaded to your warehouse. 6 years is a nice amount to fully observe trends like growth, seasonality, and buyer personas that exist in the data. Uploading this amount of data requires a few extra steps, but we'll walk you through them. If you have a preferred way of loading CSVs into your warehouse or an S3 bucket, that will also work just fine, the generated data is just CSV files.

> [!TIP]
> If you'd like to explore further on the command line, but are a little intimidated by the terminal, we've included configuration for a _task runner_ called, fittingly, `task`. It's a simple way to run the commands you need to get started with dbt. You can install it by following the instructions [here](https://taskfile.dev/#/installation). We'll call out the `task` based alternative to each command below that provides an 'easy button'. It's a useful tool to have installed regardless.

1. Create a `profiles.yml` file in the root of your project. This file is already `.gitignore`d so you can keep your credentials safe. If you'd prefer you can instead set up a `profiles.yml` file at the `~/.dbt/profiles.yml` path instead to be extra sure you don't accidentally commit the file.

2. [Add a profile for your warehouse connection in this file](https://docs.getdbt.com/docs/core/connect-data-platform/connection-profiles#connecting-to-your-warehouse-using-the-command-line) and add this configuration to your `dbt_project.yml` file as a top-level key called `profile` e.g. `profile: my-profile-name`.

> [!IMPORTANT]
> If you do decide to use `task` there is a super-task (`task load`) that will do all of the below steps for you. Just run `task load YEARS=[integer of years to generate] DB=[name of warehouse]` e.g. `task YEARS=4 DB=bigquery` or `task YEARS=7 DB=redshift` etc to perform all the commands necessary to generate and seed the data once your `profiles.yml` file is set up.

3. Create a new virtual environment in your project (I like to call mine `.venv`) and activate it, then install the project's dependencies in it. This will install the `jafgen` tool which you can use to generate the larger datasets. Then install `dbt-core` and your warehouse's adapter. You can do this manually or with `task`:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
python3 -m pip install dbt-core dbt-[your warehouse adapter] # e.g. dbt-bigquery
```

**OR**

```bash
task venv
task install DB=[name of warehouse] # e.g. task install DB=bigquery
```

> [!NOTE]
> Because you have an active virtual environment, this new install of `dbt` should take precedence in your [`$PATH`]($PATH`). If you're not familiar with the `PATH` environment variable, just think of this as the order in which your computer looks for commands to run. What's important is that it will look in your active virtual environment first, so when you run `dbt`, it will use the `dbt` you just installed in your virtual environment.

5. Run `jafgen` and `seed` the data it generates.

To generate 6 years of data:

```bash
jafgen 6
rm -rf seeds/jaffle-data
mv jaffle-data seeds
dbt seed --full-refresh --vars '{"load_source_data": true}'
```

**OR**

```bash
task gen YEARS=6
task seed
```

6. Remove the `jaffle-data` folder, then uninstall the temporary dbt Core installation. Again, this was to allow you to seed the large data files. You can then delete your `profiles.yml` file and the configuration in your `dbt_project.yml` file. You should also delete the `jaffle-data` path from the `seeds:` config in your `dbt_project.yml`.

```bash
rm -rf seeds/jaffle-data
python3 -m pip uninstall dbt-core dbt-[your warehouse adapter] # e.g. dbt-bigquery
```

**OR**

```bash
task clean
```

You now have a much more interesting and expansive dataset in your `raw` schema to build with! You should now run a `dbt build` to build the project with the new data.

### 🔍 Pre-commit and SQLFluff

There's an optional tool included with the project called `pre-commit`.

[pre-commit](https://pre-commit.com/) automatically runs a suite of of processes on your code, like linters and formatters, when you commit. If it finds an issue and updates a file, you'll need to stage the changes and commit them again (the first commit will not have gone through because pre-commit found and fixed an issue). The outcome of this is that your code will be more consistent automatically, and everybody's changes will be running through the same set of processes. We recommend it for any project.

You can see the configuration for pre-commit in the `.pre-commit-config.yaml` file. It's installed as part of the project's `requirements.txt`, but you'll need to opt-in to using it by running `pre-commit install`. This will install _git hooks_ which run when you commit. You can also run the checks manually with `pre-commit run --all-files` to see what it does without making a commit.

At present the following checks are run:

- `ruff` - an incredibly fast linter and formatter for Python, in case you add any Python models
- `check-yaml` - which validates YAML files
- `end-of-file-fixer` - which ensures all files end with a newline
- `trailing-whitespace` - which trims trailing whitespace from files

We have kept a `.sqlfluff` config file to show what that looks like, and to future proof the repo for when SQLFluff fully supports ClickHouse.
