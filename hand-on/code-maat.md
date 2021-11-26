# Code Maat

1. Organization
2. Team
3. Individual

## Generate evolution data

```sh
git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --no-merges --no-renames --after=<YYYY-MM-DD> --before=<YYYY-MM-DD>
```

- example

  ```sh
  git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --no-merges --no-renames --after=2018-01-01 --before=2018-07-01 > data/evo.log
  ```

## Running Code Maat

```sh
java -jar code-maat-[version].jar -h
```

### Generating a summary

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a summary
```

### Mining organizational metrics

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a revisions
```

### Mining logical coupling

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a coupling
```

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a coupling --temporal-period 1
```

> Modules that keep changing together that often over a period of time are probably related.

### Absolute churn

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a abs-churn
```

### Churn by author

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a author-churn
```

### Churn by entity

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a entity-churn
```

### Entity ownership

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a abs-churn
```

### Entity effort

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a author-churn
```

### Main developer

```sh
java -jar code-maat-1.1-SNAPSHOT-standalone.jar -l data/evo.log -c git -a main-dev
```
