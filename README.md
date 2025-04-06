# Project
resource "aws_api_gateway_rest_api" "Medical" {
  body = jsonencode({
    openapi = "3.0.1"
    info = {
      title   = "example"
      version = "1.0"
    }
    paths = {
      "/path1" = {
        get = {
          x-amazon-apigateway-integration = {
            httpMethod           = "GET"
            payloadFormatVersion = "1.0"
            type                 = "HTTP_PROXY"
            uri                  = "https://ip-ranges.amazonaws.com/ip-ranges.json"
          }
        }
      }
    }
  })

  name = "Medical"

  endpoint_configuration {
    types = ["REGIONAL"]
  }
}

resource "aws_api_gateway_deployment" "medical" {
  rest_api_id = aws_api_gateway_rest_api.medical.id

  triggers = {
    redeployment = sha1(jsonencode(aws_api_gateway_rest_api.medical.body))
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_api_gateway_stage" "medical" {
  deployment_id = aws_api_gateway_deployment.medical.id
  rest_api_id   = aws_api_gateway_rest_api.medical.id
  stage_name    = "medical"
}
