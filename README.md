## **Project Name**: `aws-dynamodb-python-crud`

---

## **Project Structure**

```
aws-dynamodb-python-crud/
├── dynamodb_table.tf             # Terraform to provision DynamoDB Table
├── lambda_function/
│   ├── lambda_function.py        # Python Lambda for CRUD Operations
│   └── requirements.txt
├── scripts/
│   ├── create_item.py            # Python Script to Add Item
│   ├── read_item.py              # Python Script to Read Item
│   ├── update_item.py            # Python Script to Update Item
│   └── delete_item.py            # Python Script to Delete Item
├── iam_role.tf                   # Terraform IAM Role for Lambda/DynamoDB
├── main.tf                       # Terraform Provider & Backend Config
├── variables.tf                  # Terraform Variables
├── outputs.tf                    # Terraform Outputs
└── README.md                     # Full Project Documentation
```

---

## **Terraform Code**

### **main.tf**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_dynamodb_table" "users" {
  name         = "Users"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "UserID"

  attribute {
    name = "UserID"
    type = "S"
  }

  tags = {
    Environment = "Dev"
  }
}
```

### **iam\_role.tf**

```hcl
resource "aws_iam_role" "lambda_exec_role" {
  name = "lambda_dynamodb_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_policy_attachment" "lambda_dynamodb_access" {
  name       = "lambda-dynamodb-access"
  roles      = [aws_iam_role.lambda_exec_role.name]
  policy_arn = "arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess"
}
```

---

## **Python Code: DynamoDB CRUD**

### **lambda\_function/lambda\_function.py**

```python
import boto3
import json

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    operation = event['operation']
    payload = event['payload']

    if operation == 'create':
        table.put_item(Item=payload)
        return {'message': 'Item created successfully'}

    elif operation == 'read':
        response = table.get_item(Key={'UserID': payload['UserID']})
        return response.get('Item', {})

    elif operation == 'update':
        table.update_item(
            Key={'UserID': payload['UserID']},
            UpdateExpression="set UserName=:n",
            ExpressionAttributeValues={':n': payload['UserName']},
            ReturnValues="UPDATED_NEW"
        )
        return {'message': 'Item updated successfully'}

    elif operation == 'delete':
        table.delete_item(Key={'UserID': payload['UserID']})
        return {'message': 'Item deleted successfully'}

    else:
        return {'message': 'Invalid Operation'}
```

### **lambda\_function/requirements.txt**

```
boto3
```

---

## **Local Python Scripts for DynamoDB**

### **scripts/create\_item.py**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

item = {
    'UserID': '1',
    'UserName': 'Atul Kamble'
}

table.put_item(Item=item)
print("Item Created:", item)
```

### **scripts/read\_item.py**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

response = table.get_item(Key={'UserID': '1'})
item = response.get('Item')
print("Item Retrieved:", item)
```

### **scripts/update\_item.py**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

response = table.update_item(
    Key={'UserID': '1'},
    UpdateExpression="set UserName=:n",
    ExpressionAttributeValues={':n': 'Atul K.'},
    ReturnValues="UPDATED_NEW"
)

print("Update Response:", response['Attributes'])
```

### **scripts/delete\_item.py**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

table.delete_item(Key={'UserID': '1'})
print("Item Deleted")
```

---

## **Deployment Steps**

### **Step 1: Terraform Deploy**

```bash
terraform init
terraform apply -auto-approve
```

### **Step 2: Python Script Execution**

```bash
pip install boto3
python scripts/create_item.py
python scripts/read_item.py
python scripts/update_item.py
python scripts/delete_item.py
```

---

## **Optional Enhancements**

1. **Lambda Deployment via Terraform** (Lambda + API Gateway)
2. **Add Secondary Indexes in DynamoDB**
3. **Connect via AWS SDK in Web App**
4. **Use SAM/CloudFormation for CI/CD**

---
