# AWS Athena Query Action

A GitHub Action to run an Athena query.

## Dependencies

- [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/)
- [Coreutils - GNU core utilities](https://www.gnu.org/software/coreutils/coreutils.html)

All dependencies are pre-installed in the `ubuntu-*` runner [images](https://github.com/actions/runner-images/tree/main?tab=readme-ov-file#available-images).

## Usage

We recommend using [GitHub's OIDC provider](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) to authenticate with AWS before running this action.

```yaml
steps:
- name: Run Athena Query
  uses: idealo/aws-athena-query-action@v1
  with:
    query-context: Catalog=AwsDataCatalog,Database=myDatabase
    query-string: >
      SELECT * FROM my_table
    output-location: s3://my-bucket/my-prefix
    output-filename: my-output.csv
    output-min-size: 10M
```

## Inputs

See [action.yml](action.yml) for more detail.

| Name              | Description                                                      | Required | Default                |
| ----------------- | ---------------------------------------------------------------- | -------- | ---------------------- |
| `query-context`   | The context within which the query executes.                     | Yes      | Catalog=AwsDataCatalog |
| `query-string`    | The SQL query statements to be executed.                         | Yes      |                        |
| `output-location` | The location in Amazon S3 where your query results are stored.   | Yes      |                        |
| `output-filename` | The desired name of the file where the query results are stored. | Yes      |                        |
| `output-min-size` | The minimum size of the output file in human-readable format.    | Yes      | 100K                   |

## Outputs

See [action.yml](action.yml) for more detail.

| Name              | Description                                                    |
| ----------------- | -------------------------------------------------------------- |
| `query-id`        | The unique identifier for the query execution.                 |
| `output-location` | The location in Amazon S3 where your query results are stored. |

## License

This project is licensed under the MIT License, see [LICENSE](LICENSE) for more information.
