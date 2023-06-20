# TestGitHub
=============
TestGitHub is a repository for showing the bare minimums of github and how to function etc
import boto3

def lambda_handler(event, context):
    sqs_client = boto3.client('sqs')
    
    response = sqs_client.receive_message(
        QueueUrl='YOUR_QUEUE_URL',
        MaxNumberOfMessages=10,  # Adjust as needed
        AttributeNames=['All'],
        MessageAttributeNames=['All']
    )
    
    if 'Messages' in response:
        for message in response['Messages']:
            message_body = message['Body']
            receipt_handle = message['ReceiptHandle']
            
            # Retrieve the custom attribute from the message
            message_attributes = message['MessageAttributes']
            custom_attribute_value = message_attributes.get('CustomAttribute', {}).get('StringValue')
            
            print(f"Message Body: {message_body}")
            print(f"Custom Attribute Value: {custom_attribute_value}")
            
            # Process the message based on the custom attribute value
            
            # Delete the message from the queue
            sqs_client.delete_message(
                QueueUrl='YOUR_QUEUE_URL',
                ReceiptHandle=receipt_handle
            )
    
    return {
        'statusCode': 200,
        'body': 'Message processing completed'
    }
