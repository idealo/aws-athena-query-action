# 🚀 AWS Athena Query Action

[![GitHub release](https://img.shields.io/github/v/release/idealo/aws-athena-query-action)](https://github.com/idealo/aws-athena-query-action/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A powerful and easy-to-use GitHub Action that executes AWS Athena queries and manages the results seamlessly within your CI/CD workflows.
Perfect for data validation, ETL processes, and automated reporting.

## ✨ Key Features

- 🔍 **Execute Athena Queries**: Run SQL queries on AWS Athena using query strings or pre-saved queries
- 📊 **Result Management**: Automatically handle query results with customizable file naming
- 🔒 **Size Validation**: Ensure query results meet minimum size requirements before proceeding
- 📥 **Download Results**: Optionally download query results to the GitHub runner for further processing
- ⚙️ **Workgroup Support**: Execute queries in specific Athena workgroups to separate applications
- 🧹 **Cleanup**: Automatic cleanup when result file size is smaller than the specified minimum size

## 🎯 Why Use This Action?

- **Automate Data Workflows**: Integrate data queries into your CI/CD pipelines
- **Validate Data Quality**: Run validation queries as part of your deployment process
- **Generate Reports**: Create automated reports from your data lake
- **ETL Process Integration**: Seamlessly incorporate Athena queries into your data processing workflows
- **Cost Optimization**: Leverage Athena's serverless architecture for cost-effective data processing

## 📋 Prerequisites

- [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/)
- [Coreutils - GNU core utilities](https://www.gnu.org/software/coreutils/coreutils.html)

All dependencies are pre-installed in the `ubuntu-*` runner [images](https://github.com/actions/runner-images/tree/main?tab=readme-ov-file#available-images).

## 🚀 Quick Start

We recommend using [GitHub's OIDC provider](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) to authenticate with AWS before running this action.

```yaml
name: Run Athena Query
on:
  workflow_dispatch:
  schedule:
  - cron: '0 6 * * *'

jobs:
  athena:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v5
      with:
        role-to-assume: arn:aws:iam::1234567890:role/GitHubActions
        aws-region: eu-central-1
    
    - name: Run Athena Query
      uses: idealo/aws-athena-query-action@v1
      with:
        query-context: Catalog=AwsDataCatalog,Database=myDatabase
        query-string: |
          SELECT * FROM my_table
          WHERE created = CURRENT_DATE
        output-location: s3://my-bucket/query-results
        output-filename: results.csv
```

## 📚 Usage Examples

### Basic Query Execution

```yaml
- name: Run Simple Query
  uses: idealo/aws-athena-query-action@v1
  with:
    query-context: Catalog=AwsDataCatalog,Database=analytics
    query-string: SELECT * FROM user_events LIMIT 100
    output-location: s3://analytics-results/user-events
    output-filename: user_events_sample.csv
```

### Using Saved Queries

```yaml
- name: Execute Saved Query
  uses: idealo/aws-athena-query-action@v1
  with:
    query-id: 1c345f78-1c34-1a34-1234-123ddd89012
    query-context: Catalog=AwsDataCatalog,Database=reporting
    output-location: s3://reports-bucket/monthly
    output-filename: monthly_report.csv
    output-min-size: 10M
```

### Custom Workgroup Setup

```yaml
- name: Run Query in Custom Workgroup
  uses: idealo/aws-athena-query-action@v1
  with:
    query-context: Catalog=AwsDataCatalog,Database=analytics
    query-workgroup: custom-workgroup
    query-string: |
      SELECT customer_id, SUM(order_amount) as total_spent
      FROM orders 
      WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
      GROUP BY customer_id
      ORDER BY total_spent DESC
    output-location: s3://analytics-results/customer-analysis
    output-filename: top_customers_30d.csv
    output-min-size: 5M
```

## 🔧 Configuration

### 📥 Inputs

See [action.yml](action.yml) for complete details.

| Name                | Description                                                                | Required | Default   |
| ------------------- | -------------------------------------------------------------------------- | -------- | --------- |
| `query-id`          | The unique identifier of the saved query.                                  | No       | -         |
| `query-string`      | The SQL query statements to be executed.                                   | No       | -         |
| `query-context`     | The context within which the query executes.                               | Yes      | -         |
| `query-workgroup`   | The workgroup to use for the query.                                        | No       | `primary` |
| `output-location`   | The location in Amazon S3 where your query results are stored.             | Yes      | -         |
| `output-filename`   | The desired name of the file where the query results are stored.           | Yes      | -         |
| `output-min-size`   | The minimum size of the output file (e.g., `1K`, `1M`, `1G`).              | No       | -         |
| `download-location` | The location on the runner machine where the query results are downloaded. | No       | -         |
| `download-filename` | The desired name of the file where the query results are downloaded.       | No       | -         |

**Note**: Either `query-id` or `query-string` must be provided. If `query-id` is provided, the `query-string` will be overridden with the SQL statements from the saved query.

### 📤 Outputs

| Name              | Description                                                    |
| ----------------- | -------------------------------------------------------------- |
| `query-id`        | The unique identifier of the query execution.                  |
| `output-location` | The location in Amazon S3 where your query results are stored. |

## 🛠️ Advanced Configuration

### Query Context Format

The `query-context` parameter should follow this format:
```
Catalog=<catalog-name>,Database=<database-name>
```

Example:
```yaml
query-context: Catalog=AwsDataCatalog,Database=my_analytics_db
```

### Output Size Validation

The action validates that query results meet the minimum size requirement specified in `output-min-size`.

Supported formats:
- `1024` (bytes)
- `1K` (1 kilobyte)
- `1M` (1 megabyte) 
- `1G` (1 gigabyte)

### File Management

- Results are automatically moved to your specified filename in S3
- Metadata files are cleaned up automatically
- Failed queries with insufficient output size are cleaned up to prevent storage costs

## 🔐 Security Best Practices

1. **Use OIDC Authentication**: Prefer GitHub's OIDC provider over long-lived access keys
2. **Least Privilege**: Grant only necessary permissions to your IAM role
3. **Secure S3 Buckets**: Ensure your output S3 buckets have appropriate access controls
4. **Query Validation**: Review SQL queries for potential security issues before execution

### Required AWS Permissions

Your IAM role needs the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "athena:StartQueryExecution",
        "athena:GetQueryExecution",
        "athena:GetNamedQuery",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "glue:GetDatabase",
        "glue:GetTable",
        "glue:GetPartitions"
      ],
      "Resource": "*"
    }
  ]
}
```

## 🐛 Troubleshooting

### Common Issues

**Query Execution Timeout**
- Consider optimizing your SQL query for better performance
- Partition your data to improve query performance

**Insufficient Output Size**
- The action fails if query results are smaller than `output-min-size` (human-readable format)
- Adjust the minimum size requirement or remove it from your configuration to skip this check

**S3 Access Issues**
- Verify IAM permissions for S3 bucket access
- Ensure the S3 bucket exists and is in the correct region

**Authentication Errors**
- Confirm AWS credentials are properly configured
- Check that your IAM role has the required Athena and S3 permissions

## 🤝 Contributing

We welcome contributions! Please feel free to submit issues and enhancement requests.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📞 Support

- 📖 [Documentation](https://github.com/idealo/aws-athena-query-action)
- 🐛 [Issue Tracker](https://github.com/idealo/aws-athena-query-action/issues)
- 💬 [Discussions](https://github.com/idealo/aws-athena-query-action/discussions)

## 🏢 About idealo

This action is developed and maintained by [idealo](https://github.com/idealo), one of Europe's leading price comparison platforms.
We're committed to open source and building tools that help developers work more efficiently with cloud technologies.

## 📄 License

This project is licensed under the MIT License, see the [LICENSE](LICENSE) file for more information.

---

⭐ **Found this action helpful?** Give it a star and share it with your team!
