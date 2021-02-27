# rjp - Reusable Jenkins Pipeline

Inspired by [MPL](https://github.com/griddynamics/mpl) this library supports
designing reusable, clean, and tested [Jenkins](https://www.jenkins.io/) pipelines. In
contrast to [MPL](https://github.com/griddynamics/mpl) rjp provides just
infrastructure, but no ready-to-use pipelines or pipeline elements, because
they are technology-specifc.

## Basic idea

A [Jenkins pipeline](https://www.jenkins.io/doc/book/pipeline/syntax/) consist
of [stages](https://www.jenkins.io/doc/book/pipeline/syntax/#stages), and
[stages](https://www.jenkins.io/doc/book/pipeline/syntax/#stages) use
[steps](https://www.jenkins.io/doc/book/pipeline/syntax/#steps).

[Set up](#Installation) rjp as
[Global Shared Library](https://www.jenkins.io/doc/book/pipeline/shared-libraries/#global-shared-libraries)
and [set up](#Installation) your own
[Global Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/#global-shared-libraries),
which contain common pipelines, stages, and steps.

Depending on your needs per component: use, configure, or override
pipelines, stages, and steps.

Three levels of configuration scopes are available, ordered by highest
precedence:
* component-specific
* team-specific
* organization global

## Examples
### Define a common pipeline
In directory `vars` of your own
[Global Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/#global-shared-libraries)
create a file called `YourSharedLibrary.groovy`, containing:

```
import org.rknuus.rjp.Configuration

def call(final Configuration customized_config = [:]) {
  final Configuration default_config = [
    shared_by_multiple_stages: 'bar',
    stages: [
      MyStage: [
        stage_specific_configuration: 'baz',
      ],
    ],
  ]
  final Configuration config = Configuration.merge(
    base: default_config, overlay: customized_config)

  pipeline {
    // ...
    stages {
      // ...
      stage('MyStage') {
        when { expression { currentStageEnabled() } }
        steps { executeStage(config) }
      }
      // ...
    }
  }
}
```

### Use a pipeline 'as is'
In your [`Jenkinsfile`](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/) or [Jenkins Pipeline Script](https://www.jenkins.io/pipeline/getting-started-pipelines/#approaches-to-defining-pipeline-script)
of the component:

```
@Library('YourSharedLibrary') _  // TODO(KNR): what's the point of _???
YourSharedPipeline()
```

By not passing any configuration to the pipeline you will use the default
configuration of that pipeline.

### To apply component-specific configuration values
In your [`Jenkinsfile`](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/) or [Jenkins Pipeline Script](https://www.jenkins.io/pipeline/getting-started-pipelines/#approaches-to-defining-pipeline-script)
of the component:

```
@Library('YourSharedLibrary') _

YourSharedPipeline([
  shared_by_multiple_stages: 'bar',
  stages: [
    MyStage: [
      stage_specific_configuration: 'baz',
    ],
  ],
])
```

This will overwrite default configuration values of `YourSharedPipeline` by
the component-specific configuration values. Configuration values used by
multiple stages like `shared_by_multiple_stages` reside in the outer most
level. Stage-specific ones like `stage_specific_configuration` in the scope of
the respective stage (`MyStage` in the above example).

### To use a component-specific pipeline
In directory `.jenkins` create a file `YourComponentPipeline.groovy` and in your
[`Jenkinsfile`](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/) or
[Jenkins Pipeline Script](https://www.jenkins.io/pipeline/getting-started-pipelines/#approaches-to-defining-pipeline-script)
of the component:

```
@Library('YourSharedLibrary') _

YourComponentPipeline()
```

### To override a stage
In directory `.jenkins/stages` create a file `MyStage.groovy`:

```
// implement custom stage logic
MyStep(5)

// optionally, if you want to call the original stage (like super):
executeBaseStage(CONFIG)

// implement some more custom stage logic
MyStep(7)
```

### To override a step
%%TODO(KNR): not sure whether that's possible without creating another shared library%%
In directory `.jenkins/vars` create a file `MyStep.groovy`:

```
def call(parameters) {
  // implement step
}
```

## Documentation

## Installation

### Dependencies

## Contribution
Beware, that the author is neither a Java nor a Groovy crack. So be gentle, please.

## License
Licensed under GPLv3. See accompanying `LICENSE` file.
