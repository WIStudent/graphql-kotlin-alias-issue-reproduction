# graphql-kotlin alias issue reproduction

This project is a minimal reproduction of an [issue](https://github.com/ExpediaGroup/graphql-kotlin/issues/1760) with [graphql-kotlin](https://github.com/ExpediaGroup/graphql-kotlin)
where aliases are not considered sufficiently during the generation of result classes.

Take the following schema and query:
```graphql
# schema.graphql
type Artifact {
    artifactName: String!
    someValue: String!
}

type Component {
    componentName: String!
    artifact(artifactName: String!): Artifact
}

type Bundle {
    bundleId: String!
    component(componentName: String!): Component
}

type Query {
    getBundle(bundleId: String!): Bundle
}
```

```graphql
# GetBundleQuery.graphql
query GetBundleQuery($bundleId: String!) {
    getBundle(bundleId: $bundleId) {
        bundleId
        componentFoo: component(componentName: "foo") {
            componentName
            artifactLorem: artifact(artifactName: "lorem") {
                artifactName
                someValue
            }
        }
        componentBar: component(componentName: "bar") {
            componentName
            artifactIpsum: artifact(artifactName: "ipsum") {
                artifactName
                someValue
            }
        }
    }
}
```

Running `mvn clean compile` generates the following classes inside the target directory:

```kotlin
// Bundle.kt
@Generated
public data class Bundle(
    public val bundleId: String,
    public val componentFoo: Component? = null,
    public val componentBar: Component? = null,
)
```

```kotlin
// Component.kt
@Generated
public data class Component(
    public val componentName: String,
    public val artifactLorem: Artifact? = null,
)
```

```kotlin
// Artifact.kt
@Generated
public data class Artifact(
    public val artifactName: String,
    public val someValue: String,
)
```

The generated `Bundle` class uses the `Component` class for both `componentFoo` and `componentBar` fields. The
`Component` class only contains the `artifactLorem` field, making it impossible to access the `artifactIpsum` field
on the `componentBar` component.

Instead, graphql-kotlin should generate something like this:

```kotlin
// Bundle.kt
@Generated
public data class Bundle(
    public val bundleId: String,
    public val componentFoo: Component1? = null,
    public val componentBar: Component2? = null,
)
```

```kotlin
// Component1.kt
@Generated
public data class Component1(
    public val componentName: String,
    public val artifactLorem: Artifact? = null,
)
```

```kotlin
// Component2.kt
@Generated
public data class Component2(
    public val componentName: String,
    public val artifactIpsum: Artifact? = null,
)
```

```kotlin
// Artifact.kt
@Generated
public data class Artifact(
    public val artifactName: String,
    public val someValue: String,
)
```
