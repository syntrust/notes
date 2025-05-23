# Setting Up Email Notifications for Amazon S3 Bucket Events

To enable email notifications for an S3 bucket, first create an SNS topic, then add email subscriptions, and finally configure event notifications on the S3 bucket.

### Create an SNS topic

1.  Open the [Amazon SNS console](https://console.aws.amazon.com/sns/home).
2.  From the navigation pane, choose **Topics**, and then choose **Create topic**.
3.  For **Type**, choose **Standard**.
4.  For **Name**, enter a name for your topic.
5.  For **Access Policy**, choose **Advanced**, and replace the JSON with:
```json
{
    "Version": "2008-10-17",
    "Id": "__default_policy_ID",
    "Statement": [
        {
            "Sid": "__default_statement_ID",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SNS:Publish",
                "SNS:RemovePermission",
                "SNS:SetTopicAttributes",
                "SNS:DeleteTopic",
                "SNS:ListSubscriptionsByTopic",
                "SNS:GetTopicAttributes",
                "SNS:AddPermission",
                "SNS:Subscribe"
            ],
            "Resource": ""
        }
    ]
}
```
6.  Click **Create topic**.

#### Add Email Subscriptions

1.  On the **Subscriptions** tab of the SNS topic created, choose **Create subscription**.
2.  For **Protocol**, choose **Email**.
3.  For **Endpoint**, enter the email address where you want to receive the notifications.
4.  Click **Create subscription**.
5. .You will receive a subscription confirmation email at the email address that you entered. Choose **Confirm subscription** in the email.

### Set Event Notifications for S3

1.  In the **Properties** page of your S3 bucket, find **Event notifications**, and click **Create event notification** button.
2. For **Event name**, enter a name for your evnet.
3. For **Event types**, select event types you want to receive notifications.
4. For **Destination**, select **SNS topic**.
5. For **Specify SNS topic**, choose **Choose from your SNS topics**, and choose SNS topic that you just created from the **SNS topic** dropdown list.
6. Click **Save changes**.
