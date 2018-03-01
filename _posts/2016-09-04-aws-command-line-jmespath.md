---
title: "AWS: CLI with JMESPath"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - AWS
  - JMESPath
---
There have been quite a few scripts written that use the [Amazon Web Services](https://aws.amazon.com/) (AWS) [command line interface](https://aws.amazon.com/cli/) (CLI) to query the state of objects in AWS.

AWS will generally return results back in JSON format, the majority of the time we just want a single value or subset of values.
<!--more-->

To get a value or subset of values I've seen various solutions to extract the specific part of the JSON that is required, often using [jq](https://stedolan.github.io/jq/) but you can equally use your favourite dynamic language or developer tool.

By way of example the script below extracts the status from a running Cloud formation using Node to parse the JSON.
```
while [ "$STATUS" != "UPDATE_COMPLETE" ]
do
    tput cuu 1 && tput el
    echo ${MESSAGE}
    sleep 60
    MESSAGE="$MESSAGE."
    STATUS=$(node -pe 'JSON.parse(process.argv[1]).Stacks[0].StackStatus' "$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${AWS_REGION})")
done
```

## AWS Built in filtering 
Some of the AWS CLI commands can send back a reduced payload of a single item or collection of items. To send back a specific item or subset the CLI commands have a query parameter that adheres to a [JMESPath](http://jmespath.org/). The JMESPath specifies which part or parts of the JSON object are required; the filtering is performed on the server side.

Now given that we want to extract the status from the running cloudformation we can pass in the query and JMESPath.
```
aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${AWS_REGION} --output text --query "Stacks[0].StackStatus"
```

For this to work you must know what the expected JSON response is and the path to your element(s).

Once you start to play with JMESPath you will find that it is quite powerful and if supported fully can do much more than simply filter your JSON. The code below will query the [Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) returning back the ImageId of the latest image (sorted by CreationDate).
```
aws ec2 describe-images \
    --filters Name=virtualization-type,Values=hvm \
            Name=architecture,Values=x86_64 \
            Name=state,Values=available \
            Name=image-type,Values=machine \
            Name=root-device-type,Values=ebs \
            Name=name,Values=\"$1\" \
            Name=is-public,Values=true \
    --region eu-west-1 \
    --output text \
    --query "reverse(sort_by(Images, &CreationDate))[0].[ImageId]"
}
```

Hopefully next time I write a script that queries an Amazon Web Service to retrieve a single value I will remember to use JMESPath.