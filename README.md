{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "Compose": {
                "inputs": "@{body('Select')}",
                "runAfter": {
                    "Select": [
                        "Succeeded"
                    ]
                },
                "type": "Compose"
            },
            "Compose_2": {
                "inputs": "@Replace(outputs('Compose'),'\"boundingBox\"','\"text\"')",
                "runAfter": {
                    "Compose": [
                        "Succeeded"
                    ]
                },
                "type": "Compose"
            },
            "Compose_3": {
                "inputs": "@split(outputs('Compose_2'),'\"text\":')",
                "runAfter": {
                    "Compose_2": [
                        "Succeeded"
                    ]
                },
                "type": "Compose"
            },
            "Delay": {
                "inputs": {
                    "interval": {
                        "count": 20,
                        "unit": "Second"
                    }
                },
                "runAfter": {
                    "URI_Location": [
                        "Succeeded"
                    ]
                },
                "type": "Wait"
            },
            "For_each": {
                "actions": {
                    "Add_a_row_into_a_table": {
                        "inputs": {
                            "body": {
                                "Key": "@{outputs('Compose_4')}"
                            },
                            "host": {
                                "connection": {
                                    "name": "@parameters('$connections')['excelonline']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/codeless/v1.2/drives/me/items/@{encodeURIComponent('92AA56BA4DF1B1B1!38045')}/workbook/tables/@{encodeURIComponent('{yTD462CC-75F0-4453-A428-F24540B42BA2}')}/rows",
                            "queries": {
                                "source": "me"
                            }
                        },
                        "metadata": {
                            "92AA56BA4DF1B1B1!38045": "/AI/Invoices_1654248.xlsx"
                        },
                        "runAfter": {
                            "Compose_4": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    },
                    "Compose_4": {
                        "inputs": "@items('For_each')",
                        "runAfter": {},
                        "type": "Compose"
                    }
                },
                "foreach": "@outputs('Compose_3')",
                "runAfter": {
                    "Compose_3": [
                        "Succeeded"
                    ]
                },
                "runtimeConfiguration": {
                    "concurrency": {
                        "repetitions": 1
                    }
                },
                "type": "Foreach"
            },
            "HTTP": {
                "inputs": {
                    "body": "@binary(triggerBody())",
                    "headers": {
                        "Content-Length": "",
                        "Content-Type": "application/pdf",
                        "Host": "https://nedermanai.cognitiveservices.azure.com/",
                        "Ocp-Apim-Subscription-Key": "08899759bdc6415590d757c7186eafd5"
                    },
                    "method": "POST",
                    "uri": "https://nedermanai.cognitiveservices.azure.com//formrecognizer/v2.1-preview.2/custom/models/ffed1df1-4886-4569-aef1-59f32b98e312/analyze"
                },
                "runAfter": {},
                "type": "Http"
            },
            "HTTP_2": {
                "inputs": {
                    "headers": {
                        "Content-Type": "application/pdf",
                        "Ocp-Apim-Subscription-Key": "08899759bdc6415590d757c7186et4e4"
                    },
                    "method": "GET",
                    "uri": "@variables('result')"
                },
                "runAfter": {
                    "Delay": [
                        "Succeeded"
                    ]
                },
                "type": "Http"
            },
            "Initialize_variable": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Invoice",
                            "type": "array",
                            "value": "@variables('InvoiceObject').analyzeResult.pageResults"
                        }
                    ]
                },
                "runAfter": {
                    "Initialize_variable_2": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Initialize_variable_2": {
                "inputs": {
                    "variables": [
                        {
                            "name": "InvoiceObject",
                            "type": "object",
                            "value": "@body('HTTP_2')"
                        }
                    ]
                },
                "runAfter": {
                    "HTTP_2": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Parse_JSON": {
                "inputs": {
                    "content": "@string(outputs('HTTP')['headers'])",
                    "schema": {
                        "properties": {
                            "Content-Length": {
                                "type": "string"
                            },
                            "Date": {
                                "type": "string"
                            },
                            "Operation-Location": {
                                "type": "string"
                            },
                            "Strict-Transport-Security": {
                                "type": "string"
                            },
                            "apim-request-id": {
                                "type": "string"
                            },
                            "x-content-type-options": {
                                "type": "string"
                            },
                            "x-envoy-upstream-service-time": {
                                "type": "string"
                            }
                        },
                        "type": "object"
                    }
                },
                "runAfter": {
                    "HTTP": [
                        "Succeeded"
                    ]
                },
                "type": "ParseJson"
            },
            "Select": {
                "inputs": {
                    "from": "@variables('Invoice')",
                    "select": {
                        "": "@item()['tables']"
                    }
                },
                "runAfter": {
                    "Initialize_variable": [
                        "Succeeded"
                    ]
                },
                "type": "Select"
            },
            "URI_Location": {
                "inputs": {
                    "variables": [
                        {
                            "name": "result",
                            "type": "string",
                            "value": "@body('Parse_JSON')?['Operation-Location']"
                        }
                    ]
                },
                "runAfter": {
                    "Parse_JSON": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "When_a_file_is_created": {
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['onedrive']['connectionId']"
                        }
                    },
                    "method": "get",
                    "path": "/datasets/default/triggers/onnewfilev2",
                    "queries": {
                        "folderId": "92AA56BA4DF1B1B1!37887",
                        "includeSubfolders": false,
                        "inferContentType": true,
                        "simulate": false
                    }
                },
                "metadata": {
                    "92AA56BA4DF1B1B1!37887": "/AI2"
                },
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 3
                },
                "type": "ApiConnection"
            }
        }
    },
    "parameters": {
        "$connections": {
            "value": {
                "excelonline": {
                    "connectionId": "/subscriptions/a614cfcf-d6c1-4886-9c08-89446fdcb1ec/resourceGroups/trainingai/providers/Microsoft.Web/connections/excelonline",
                    "connectionName": "excelonline",
                    "id": "/subscriptions/a614cfcf-d6c1-4886-9c08-89446fdcb1ec/providers/Microsoft.Web/locations/uksouth/managedApis/excelonline"
                },
                "onedrive": {
                    "connectionId": "/subscriptions/a614cfcf-d6c1-4886-9c08-89446fdcb1ec/resourceGroups/trainingai/providers/Microsoft.Web/connections/onedrive-1",
                    "connectionName": "onedrive-1",
                    "id": "/subscriptions/a614cfcf-d6c1-4886-9c08-89446fdcb1ec/providers/Microsoft.Web/locations/uksouth/managedApis/onedrive"
                }
            }
        }
    }
}
