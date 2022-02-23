## AWS-IPRanges-API

[Serverless](https://aws.amazon.com/serverless/) web portal based on [Amazon API Gateway](https://aws.amazon.com/api-gateway/) and [AWS Lambda](https://aws.amazon.com/lambda/) that outputs IP Prefixes from AWS ip-ranges.json for use by firewalls and other applications

## Description
AWS [publishes](https://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html) its IP ranges in json format through [ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json). The IP prefixes are commonly used by firewalls for network access control. A common use case is [CloudFront](https://aws.amazon.com/cloudfront/) with origin as customer's on-premise web server where customers [protect origin](https://docs.aws.amazon.com/whitepapers/latest/secure-content-delivery-amazon-cloudfront/protecting-your-origin-by-allowing-access-to-cloudfront-only.html) by only allowing CloudFront (and if in use Route 53 Health Checks) IP address ranges inbound access to on-premise web server using firewall policies.

Typically, the firewall administrator will extract out required IP prefixes from ip-ranges.json for use in their firewall rules. The IP prefixes need to be updated whenever there are changes and customer can subscribe to [AWS IP address ranges notifications](https://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html#subscribe-notifications)

This project makes IP prefixes available as web feeds for dynamic updates by firewalls. Users have the option to filter the IP prefixes by service, region and network border group and in combined IPv4 and IPv6, IPv4 only or IPv6 only format. Entire solution is serverless, and can be deployed via a single CloudFormation file. 

## Architecture Diagram
<img width="1340" alt="image" src="https://user-images.githubusercontent.com/88474310/155283397-b34594ea-213d-4b8f-b391-6081087f1743.png">


## Deployment via CloudFormation
To deploy via AWS [CloudFormation console](https://console.aws.amazon.com/cloudformation), use [Create Stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) to upload `template.yaml` file. The template has default parameters values that you can change
- `awsServices`: Name of AWS services to return by root URL separated by commas. Default is **CLOUDFRONT_ORIGIN_FACING,ROUTE53_HEALTHCHECKS**
- `lambdaFunctionName`: Lambda function name. Default is **IPRanges-2-apiGW**
- `pythonRuntime`: Python 3 runtime version. Default is **python3.9**
- `cpuArchitecture`: Instruction set architecture (x86_64 or ARM64). Default is **x86_64**

Go to Outputs tab to get `apiGatewayInvokeURL` value (in the form https://\<VALUE\>.execute-api.\<REGION\>.amazonaws.com) for use by firewall. Refer to [Output options](https://github.com/aws-samples/aws-ipranges-api#output-options) below for more details.

  
## Firewall Setup
Firewalls that supports external intel/threat feed for IP prefixes updates include (but not limited to)
- Palo Alto: [External Dynamic List](https://docs.paloaltonetworks.com/pan-os/10-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/external-dynamic-list.html) 
- Fortigate: [External Block List](https://docs.fortinet.com/document/fortigate/7.0.5/administration-guide/891236/external-blocklist-policy)
- pfSense: [URL Table Aliases](https://docs.netgate.com/pfsense/en/latest/firewall/aliases.html#url-table-aliases)
- OPNsense: [URL Tables (IPs) Aliases](https://docs.opnsense.org/manual/aliases.html)

Refer to vendor website documentation for configuration steps.
For Check Point users, AWS IP prefixes are available through [Updatable Objects](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk131852)

## Output options
Use different URLs to return IP prefixes. Example
  - `/` : return CloudFront origin facing and R53 Health Checks  prefixes, customizable via `SERVICES` Lambda environment variable
  - `/SERVICE` : listing of available services
  - `/REGION` : listing of available regions
  - `/NETWORK` : listing of network border groups which is a unique set of Availability Zones or Local Zones from where AWS advertises IP addresses
  - `/SERVICE/<SERVICE>` : prefixes for specific SERVICE, e.g. /SERVICE/S3
  - `/SERVICE/<SERVICE>/<REGION>` : prefixes for specific SERVICE and REGION, e.g. /SERVICE/EC2/us-east-1
  - `/REGION/<REGION>` : prefixes for specific REGION, e.g. /REGION/ap-southeast-1
  - `/NETWORK/<NETWORK>` : prefixes for specific network border group, e.g. /NETWORK/us-east-1-nyc-1

Append `/IPv4.txt` or `/IPv6.txt` to limit IP prefixes to IPv4 or IPv6 respectively

## API Gateway Customisation
Refer to [Amazon API Gateway documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) for HTTP API customisation options. Some examples include
- [Private integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-private.html) for use in a VPC
- [Setting up custom domain names](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-custom-domain-names.html)
- [Configuring mutual TLS authentication](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-mutual-tls.html)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

