# Amazon Connect AI Agents - CloudWatch Logs Setup (CloudFormation)

Automate CloudWatch Logs setup for Amazon Connect AI Agents with a single CloudFormation template. No more manual configuration—deploy in 3 minutes instead of 30.

[![AWS CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazon-aws)](https://aws.amazon.com/cloudformation/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 🎯 What This Does

This CloudFormation template automatically configures CloudWatch Logs for your Amazon Connect AI Agents, enabling you to:

- 📊 Monitor AI Agent interactions in real-time
- 🔍 Troubleshoot issues with detailed event logs
- 📈 Analyze LLM invocations, recommendations, and intent detection
- 🎯 Track session-level AI Agent performance

## ✨ Features

- **One-Click Deployment**: Single CloudFormation stack creates all required resources
- **Automatic IAM Configuration**: Properly scoped roles and policies with least-privilege access
- **CloudWatch Logs Delivery**: Automated setup of delivery source, destination, and delivery
- **Easy Cleanup**: Delete the stack to remove all resources
- **Cost-Effective**: Pay only for what you use (~$2-5/month for 10,000 interactions)

## 📋 Prerequisites

Before deploying, ensure you have:

- ✅ Amazon Connect instance with AI Agents enabled
- ✅ Your **Assistant ARN** (from Amazon Connect console → Agent workspace → AI agents)
- ✅ AWS CLI configured or access to CloudFormation console
- ✅ IAM permissions to create CloudFormation stacks with IAM resources

## 🚀 Quick Start

### Option 1: Deploy via AWS Console

1. **Get your Assistant ARN**:
   - Open Amazon Connect console
   - Navigate to **Agent workspace** → **AI agents**
   - Copy the ARN (format: `arn:aws:wisdom:region:account-id:assistant/assistant-id`)

2. **Deploy the stack**:
   - Download [`enable-ai-agent-cloudwatch-logs.yaml`](enable-ai-agent-cloudwatch-logs.yaml)
   - Open [CloudFormation Console](https://console.aws.amazon.com/cloudformation)
   - Click **Create stack** → **With new resources**
   - Upload the template file
   - Fill in the **AssistantArn** parameter
   - Accept defaults or customize other parameters
   - Check **"I acknowledge that AWS CloudFormation might create IAM resources"**
   - Click **Submit**

3. **Wait 2-3 minutes** for stack creation to complete

4. **View logs**:
   - Go to the **Outputs** tab
   - Click the **ConsoleUrl** to view your logs

### Option 2: Deploy via AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name ai-agent-cloudwatch-logs \
  --template-body file://enable-ai-agent-cloudwatch-logs.yaml \
  --parameters \
    ParameterKey=AssistantArn,ParameterValue=arn:aws:wisdom:REGION:ACCOUNT:assistant/ASSISTANT-ID \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

Monitor deployment:

```bash
aws cloudformation wait stack-create-complete \
  --stack-name ai-agent-cloudwatch-logs \
  --region us-east-1
```

## 📊 Viewing Logs

### CloudWatch Console

1. Navigate to **CloudWatch** → **Logs** → **Log groups**
2. Find `/aws/connect/ai-agents` (or your custom log group name)
3. View log streams for AI Agent events

### CloudWatch Logs Insights Queries

**View all AI Agent events:**
```sql
fields @timestamp, event_type, session_id, assistant_id
| sort @timestamp desc
| limit 100
```

**Find LLM invocations:**
```sql
fields @timestamp, event_type, model_id, prompt, completion
| filter event_type = "TRANSCRIPT_LARGE_LANGUAGE_MODEL_INVOCATION"
| sort @timestamp desc
```

**Track a specific session:**
```sql
fields @timestamp, event_type, recommendation
| filter session_id = "YOUR-SESSION-ID"
| sort @timestamp asc
```

**Analyze recommendations:**
```sql
fields @timestamp, recommendation_id, recommendation, is_recommendation_useful
| filter event_type = "TRANSCRIPT_RECOMMENDATION"
| sort @timestamp desc
```

## ⚙️ Configuration Parameters

| Parameter | Description | Default | Required |
|-----------|-------------|---------|----------|
| `AssistantArn` | Amazon Connect Assistant ARN | - | ✅ Yes |
| `DeliverySourceName` | Name for delivery source | `connect-ai-agent-delivery-source` | No |
| `LogGroupName` | CloudWatch Log Group name | `/aws/connect/ai-agents` | No |
| `LogRetentionDays` | Log retention period | `30` | No |
| `OutputFormat` | Log format (json/plain/w3c/raw/parquet) | `json` | No |
| `Environment` | Environment tag | `Production` | No |
| `CostCenter` | Cost center for billing | `ContactCenter` | No |

## 🏗️ What Gets Created

The CloudFormation stack creates:

1. **CloudWatch Log Group** - Stores AI Agent logs with configurable retention
2. **IAM Roles** - Least-privilege roles for CloudWatch Logs delivery
3. **Resource Policies** - Allows Wisdom service to write logs
4. **Lambda Function** - Orchestrates CloudWatch Logs Delivery API calls
5. **CloudWatch Logs Delivery** - Links Assistant to Log Group

## 🔧 Troubleshooting

### Stack Creation Fails with AccessDeniedException

**Cause**: Lambda role missing required permissions.

**Solution**: Ensure you're using the latest template version which includes:
- `iam:PassRole` permission
- `wisdom:AllowVendedLogDeliveryForResource` permission

### No Logs Appearing

**Solutions**:
1. Wait 1-2 minutes after AI Agent interactions
2. Verify AI Agent was actually invoked
3. Check delivery status:
   ```bash
   aws logs describe-deliveries --region us-east-1
   ```
4. Verify Assistant ARN is correct and in the same region

### Wrong Region Error

**Cause**: Stack deployed in different region than Assistant.

**Solution**: Deploy in the same region as your Assistant ARN:
```
arn:aws:wisdom:us-east-1:...
                 ^^^^^^^^^ Deploy stack here
```

## 💰 Cost Estimate

| Service | Pricing | Monthly Cost (10K interactions) |
|---------|---------|--------------------------------|
| CloudWatch Logs Ingestion | $0.50/GB | $1-3 |
| CloudWatch Logs Storage | $0.03/GB/month | $0.50-1.50 |
| CloudWatch Logs Insights | $0.005/GB scanned | Pay per query |
| Lambda | Free (only runs on stack create/delete) | $0 |

**Total**: ~$2-5/month for 10,000 AI Agent interactions

## 🧹 Cleanup

To remove all resources:

```bash
aws cloudformation delete-stack \
  --stack-name ai-agent-cloudwatch-logs \
  --region us-east-1
```

> ⚠️ **Warning**: This deletes the log group and all logs. Export any logs you need before deletion.

## 📚 Additional Resources

- [Amazon Connect AI Agents Documentation](https://docs.aws.amazon.com/connect/latest/adminguide/amazon-connect-ai-agents.html)
- [Monitor AI Agents with CloudWatch Logs](https://docs.aws.amazon.com/connect/latest/adminguide/monitor-ai-agents.html)
- [CloudWatch Logs Delivery API](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/Welcome.html)
- [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙋 Support

If you encounter issues:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Review CloudFormation stack events for error details
3. Check Lambda function logs in CloudWatch
4. Open an issue in this repository

## ⭐ Show Your Support

If this project helped you, please consider giving it a ⭐ star!
