# Foreman Training - Teil 9 - Hammer CLI

Foreman Config von der Kommandozeile

## Konfiguratoin

Die Konfiruration wird pro User hinterlegt.

`.hammer/cli.modules.d/foreman.yml`
    :foreman:
      # Credentials. You'll be asked for the interactively if you leave them blank here
      :username: 'admin'
      :password: '12345678'

      # Check API documentation cache status on each request
      :refresh_cache: false

      # API request timeout in seconds, set -1 for infinity
      :request_timeout: 120

## hammer Kommando

`hammer`

Beispiel:

    hammer host list
    ---|----------------------------|------------------|------------|---------------|-------------------|---------------|----------------|----------------------
    ID | NAME                       | OPERATING SYSTEM | HOST GROUP | IP            | MAC               | GLOBAL STATUS | CONTENT VIEW   | LIFECYCLE ENVIRONMENT
    ---|----------------------------|------------------|------------|---------------|-------------------|---------------|----------------|----------------------
    2  | apache.example42.training  | CentOS 7.9       |            | 10.100.10.104 | 08:00:27:75:19:a8 |  Warning       | Composite View | Produktion
    1  | foreman.example42.training | CentOS 7.9.2009  |            | 10.0.2.15     | 52:54:00:4d:77:d3 | OK            | Composite View | Produktion
    ---|----------------------------|------------------|------------|---------------|-------------------|---------------|----------------|----------------------


Weiter mit Teil 10 [Maintenance](../10_maintenance)
