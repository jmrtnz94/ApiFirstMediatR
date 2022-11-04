# ApiFirstMediatR

![CI](https://github.com/alexcampana/ApiFirstMediatR/workflows/CI/badge.svg)
[![NuGet](https://img.shields.io/nuget/vpre/ApiFirstMediatR.Generator.svg)](https://www.nuget.org/packages/ApiFirstMediatR.Generator)

Generates Controllers, DTOs and MediatR Requests from a given OpenAPI Spec file to support API First development.
Business logic implementation is handled by MediatR handlers that implement the generated MediatR Requests.

Code is generated using a Roslyn based Source Generator. To find out more about Roslyn Source Generators go here: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

Currently supports ASP.NET Core 6.0 and OpenAPI Spec version 3 and 2 in both yaml and json formats.

## Installation
```sh
dotnet add package ApiFirstMediatR.Generator
```

Register your OpenAPI spec file by adding the following to your `.csproj`:
```xml
    <ItemGroup>
        <!-- Registers the OpenAPI spec -->
        <AdditionalFiles Include="api_spec.json" />
    </ItemGroup>
```

### Hello World Example
Hello World OpenAPI spec:
```yaml
openapi: 3.0.1
info:
  title: HelloWorld API
  version: v1
paths:
  /api/HelloWorld:
    get:
      tags:
        - HelloWorld
      operationId: GetHelloWorld
      parameters: []
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HelloWorldDto'
components:
  schemas:
    HelloWorldDto:
      type: object
      properties:
        message:
          type: string
          nullable: true
security: []
```

The source generators will then generate the following based on he given OpenAPI spec file:
```csharp
// <auto-generated/>
namespace HelloWorld.Dtos
{
    public class HelloWorldDto 
    {
        public string? Message { get; set; } // TODO: Set the JsonProperty Attributes
    }
}
```
```csharp
// <auto-generated/>
#nullable enable
using MediatR;
using HelloWorld.Dtos;

namespace HelloWorld.Requests
{
    public sealed class GetHelloWorldQuery : IRequest<HelloWorldDto>
    {
    }
}
```
```csharp
// <auto-generated/>
using MediatR;
using Microsoft.AspNetCore.Mvc;
using HelloWorld.Dtos;
using HelloWorld.Requests;

namespace HelloWorld.Controllers
{
    [Route(""[controller]"")]
    public sealed class ApiController : Controller
    {
        private readonly IMediator _mediator;

        public ApiController(IMediator mediator)
        {
            _mediator = mediator;
        }

        [HttpGet(""/api/HelloWorld"")]
        public async Task<ActionResult> GetHelloWorld()
        {
            var request = new GetHelloWorldQuery
            {
            };

            var response = await _mediator.Send(request); // TODO: optionally use a CancellationToken
            return Ok(response);
        }
    }
}
```

To implement the Hello World endpoint create the following handler:
```csharp
using HelloWorld.Dtos;
using HelloWorld.Requests;
using MediatR;

namespace HelloWorld.Handlers
{
    public sealed class GetHelloWorldQueryHandler : IRequestHandler<GetHelloWorldQuery, HelloWorldDto>
    {
        public Task<HelloWorldDto> Handle(GetHelloWorldQuery query, CancellationToken cancellationToken)
        {
            // Endpoint implementation goes here
        }
    }
}
```

## Viewing the generated files
### Visual Studio
In the solution explorer expand Project -> Dependencies -> Analyzers -> ApiFirstMediatR.Generator -> ApiFirstMediatR.Generator.ApiSourceGenerator.

### Rider
In the explorer expand Project -> Dependencies -> .NET 6.0 -> Source Generators -> ApiFirstMediatR.Generator.ApiSourceGenerator

### VSCode
Add the following to your `.csproj`
```xml
    <!-- Begin VSCode Compatibility -->
    <PropertyGroup>
        <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>
        <CompilerGeneratedFilesOutputPath>Generated</CompilerGeneratedFilesOutputPath>
    </PropertyGroup>
    <ItemGroup>
        <Compile Remove="$(CompilerGeneratedFilesOutputPath)/**/*.cs" />
    </ItemGroup>

    <Target Name="CleanSourceGeneratedFiles" BeforeTargets="BeforeRebuild" DependsOnTargets="$(BeforeBuildDependsOn)">
        <RemoveDir Directories="$(CompilerGeneratedFilesOutputPath)" />
    </Target>
    <!-- End VSCode Compatibility -->
```

This will force the generators to output the generated files to the `Generated` directory, but notify the compiler to ignore the generated files and continue to use the normal Roslyn Source Generator compile process. Adding the Generated directory to your `.gitignore` is recommended.
