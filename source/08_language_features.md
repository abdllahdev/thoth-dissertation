\chapter{Key Features and Syntax}
\label{chap:lang-features}

In this chapter, we will provide a comprehensive introduction to the DSL by explaining its key features and syntax. The main goal of this chapter is to give a thorough understanding of the language's declarations and expressions. The DSL's syntax is designed around a few declarations. In each section in this chapter, we are going to explain how to use these declarations to build web applications using Thoth. For information on the language grammar defined in Backus–Naur form (BNF), please refer to Appendix [1](#appendix-1-language-grammar).

## App Configurations

The `app` declaration serves as the project's starting point and is used to define global configurations such as the app's title and authentication configuration. Every app built with the DSL must be declared, and there can only be one `app` declaration. The `app` declaration takes a JSON-like object as a value. In the `app` declaration body, only the `title` field is required. The app's `title` will appear in the browser tab. We will talk in more detail about authentication in a later section. Apps can be declared as shown in the code in Listing \ref{lst:app-declaration-example}.

\begin{lstlisting}[caption=App config example.,label={lst:app-declaration-example},language=Thoth]
app ExampleApp {
  title: "Example App"
}
\end{lstlisting}

## Data Models

Data models represent entities in the application domain and map to database tables. Models are declared using the `model` declaration. Every model must be declared with a name following the PascalCase convention. Each model must define several fields. For example, a blogging platform can define `Post` data model as shown in Listing \ref{lst:model-example}.

### Model Fields

Model fields are like columns in a database table. They define what kind of data can be stored in the table. Each field has a name, a type, an optional type modifier, and an optional attribute. The field name is a string that identifies the field within the model. The name must start with a letter and can be followed by any combination of letters, numbers, or underscores. It is common to name fields using the camelCase convention.

When defining model fields, it is necessary to specify the type of each field when defining it. There are two main categories of types: scalar types and user-defined types. The DSL has four primary scalar types: `Int`, `String`, `Boolean`, and `DateTime`. User-defined types refer to data models that are created by the user. These user-defined types can be used as types for fields to specify relations between different models in the application. In the next section, we will elaborate on the topic of relations in greater detail.

The type of field can be modified by appending either of two modifiers: `[]` to make it a list or `?` to make it optional. Optional lists are not supported, so type modifiers cannot be combined.

Attributes are also optional and can be used to provide additional information about a field. For example, you can specify that a field is unique (i.e., it cannot have duplicate values). All attributes start with the special character `@`. Fields can have one or more of the following attributes:

- **\@id**: The `@id` attribute defines a unique field that can be used to identify individual model records. A model can only have one ID field because a field that implements the `@id` attribute is similar to a `PRIMARY_KEY` field in relational databases. The ID field must be of `Int` type.

- **\@unique**: The `@unique` attribute can be used to make a field have unique values. For example, in a blogging platform, each post must have a unique slug, which is a human-readable, unique identifier used to identify a post instead of id.

- **\@default**: Scalar fields can be created with default values using the `@default` attribute. The `@default` attribute takes the default value as an argument. The default value must be a literal value of the same type as the field type, such as `5` (`Int`), `"Hello"` (`String`), `false` (`Boolean`), or `Now` (`DateTime`). `Now` is a keyword that can be used to set a timestamp of the time when a record is created.

- **\@updatedAt**: The `@updatedAt` attribute can be used with fields of type `DateTime` to automatically store the time when a record was last updated.

- **\@relation**: This attribute is used to define relation fields. In the next section, we will elaborate on the topic of relations in greater detail.

The code in Listing \ref{lst:model-example} shows how fields can be defined using the concepts explained above. The syntax for defining fields is similar to Prisma Schema Language[@prisma-schema-ref]. First, the field name is declared, which is then followed by the type, type modifier, and any number attributes. Each field must be declared on a new line, and the field name, type, and attributes must be separated using a space. The example implements a `Post` model that has an ID field, uses all the scalar types in the language, makes use of type modifiers, and shows how field attributes can be used.

\begin{lstlisting}[caption=Model example.,label={lst:model-example},language=Thoth]
model Post {
  id        Int      @id // an id field
  title     String // a `String` field that does not have any attributes
  slug      String?  @unique // a unique optional field of type `String`
  content   String
  published Boolean  @default(false) // a field with `false` as a default value
  createdAt DateTime @default(Now) // a field with a `Now` as default value
  updatedAt DateTime @updatedAt @ignore // a `DateTime` field that gets updates whenever a post record is updated and is ignored whenever posts are queried
}
\end{lstlisting}

### Relations

As mentioned before, models can define relations between each other. Relations are defined between models using user-defined types and the `@relations` attribute. The relation must be defined in both related models or the compiler will throw an error. The `@relation` attribute takes two arguments: the first is called the relation field, and the second is called the reference field. There are three types of relations: one-to-one relations, one-to-many relations, and many-to-many relations.

**One-to-One Relations**

One-to-one relations refer to relations where at most one record can be connected on both sides of the relation. For example, in the code in Listing \ref{lst:one-to-one-relation} each user is associated with zero or one profile through the relation field `profile` in the `User` model. The `user` field in the `Profile` model defines a relation with the `User` model by referencing the field `id` in the `User` model. The `userId` relation field represents the foreign key. In this case, the relation field must implement the `@unique` attribute to make sure that every user is associated with only one profile.

\begin{lstlisting}[caption=One-to-one relation example.,label={lst:one-to-one-relation},language=Thoth]
model User {
  id      Int      @id
  profile Profile?
}

model Profile {
  id     Int  @id
  user   User @relation(userId, id)
  userId Int  @unique // relation field (used in the `@relation` attribute in line 8)
}
\end{lstlisting}

**One-to-Many Relations**

One-to-many relations refer to relations where one record on one side of the relation can be connected to zero or more records on the other side. As shown in the code in Listing \ref{lst:one-to-many-relation}, each user has zero or many posts, and each post has only one user. Similar to the previous example, the `User` model defines a relation with the `Post` model via the `posts` relation field. Then, the `Post` model references the `id` field in the `User` model using the `@relation` attribute. Unlike one-to-one relations, the relation field `userId` in the model `Post` must not implement the `@unique` attribute because a user can have more than one post.

\begin{lstlisting}[caption=One-to-many relation example.,label={lst:one-to-many-relation},language=Thoth]
model User {
  id    Int    @id
  posts Post[]
}

model Post {
  id     Int  @id
  user   User @relation(userId, id)
  userId Int
}
\end{lstlisting}

**Many-to-Many Relations**

Many-to-many relations refer to relations where zero or more records on one side of the relation are connected to zero or more records on the other side. Many-to-many relations are easy to implement by defining that each model has a list of the other model. As shown in the example in Listing \ref{lst:many-to-many-relation}, The `Post` model defines a relation field `tags` with the `Tag` model. Similarly, the model `Tag` defines a relation field `posts` with the `Post` model.

\begin{lstlisting}[caption=Many-to-many relation example,label={lst:many-to-many-relation},language=Thoth]
model Post {
  id    Int   @id
  tags  Tag[]
}

model Tag {
  id    Int    @id
  posts Post[]
}
\end{lstlisting}

## Queries

The declaration of queries is a fundamental feature of the DSL that is designed to facilitate the creation of queries on data models. It offers five distinct types of queries, including `FindMany`, `FindUnique`, `Create`, `Update`, and `Delete`. Queries are defined using the `query` declaration, which is one of the declarations in the DSL that must take a type and a JSON-like object as a body. The body can implement one or more of the following entries: `where`, `search`, or `data`, based on the query type. Moreover, they must define the data model to query. This can be done using the query attribute `@model`. Furthermore, they run on the server only. Generally, query names should follow the camelCase convention.

The `where` entry is used to specify a unique field in the data model, and it is used to select an individual record using the unique field. In contrast, the `search` entry is used to filter a list of records based on the list of fields. The `data` entry is used to specify the fields that a `Create` or `Update` query can use to create a new record or update an existing one.

**FindMany Queries**

`FindMany` queries are used to read a list of records from a data model. Any `FindMany` query must implement a `search` entry that takes a list of fields that can be used to filter the records in the database. `FindMany` queries cannot implement either the `where` or the `data` entries. For example, for the `Post` model shown in Listing \ref{lst:model-example}, we can implement a `FindMany` query that returns a list of posts or filter them based on the `published` field, as shown below:

\begin{lstlisting}[caption=FindMany query example.,label={lst:find-many-query},language=Thoth]
@model(Post)
query<FindMany> getPosts {
  search: [ published ]
}
\end{lstlisting}

**FindUnique Queries**

Individual records can be read using `FindUnique` queries. `FindUnique` queries must implement the `where`, and it must take a unique field, for the query to identify the record. For instance, to get a post by id, this query can be defined as follows:

\begin{lstlisting}[caption=FindUnique query example.,label={lst:find-unique-query},language=Thoth]
@model(Post)
query<FindUnique> getPostById {
  where: id
}
\end{lstlisting}

**Create Queries**

Create queries can only implement the `data` entry, which can be used to specify the fields needed to create a new post record. The `data` entry takes two entries: `fields` a required entry that specifies non-relation fields and `relationFields` an optional entry that specifies how models should be linked with each other. Create queries must specify all the required fields in a model. For instance, consider the `Post` model in the example below, the create query for this model must specify the `title`, `content`, and `user` fields because they are all required. Any optional or list fields do not have to be specified. As well as fields that have `@default` or `@updatedAt` attributes, because records can be created using the default values.

The `Create` query shown below specifies that the query can create a post record using the `title` and `content` fields in the `fields` entry. While the `relationFields` specifies how a post record can be linked to a user using the `connect-with` expression. In this case, the `user` field is connected to the `id` field of a user record using `userId` field in a post record as a foreign key field without specifying them.

\begin{lstlisting}[caption=Create query example.,label={lst:create-query},language=Thoth]
@model(Post)
query<Create> createPost {
  data: {
    fields: [ title, content ],
    relationFields: {
      user: connect id with userId
    }
  }
}
\end{lstlisting}

**Update Queries**

A record can be updated by defining an update query. Update queries must define the `where` and `data` entries. Similar to `FindUnique` queries, the `where` entry must specify a unique field, for the query to be able to find the record that needs to be updated. Where the `data` entry takes the field that can be updated. The example below defines an update query that can update the `title`, `content`, or `published` fields of a post record.

\begin{lstlisting}[caption=Update query example,label={lst:update-query},language=Thoth]
@model(Post)
query<Update> updatePostById {
  where: id,
  data: {
    fields: [ title, content, published ]
  }
}
\end{lstlisting}

**Delete Queries**

To define a delete query, the query must only implement the `where` entry. The body of the delete query is similar to `FindUnique` queries.

\begin{lstlisting}[caption=Delete query example.,label={lst:delete-query},language=Thoth]
@model(Post)
query<Delete> deletePostById {
  where: id
}
\end{lstlisting}

## UI Components

User interfaces can be built using the `component` declaration. Components can be declared with a type, or without declaring a general UI component. There are several types of components, including fetch components (FindMany, FindUnique) and action components (Create, Update, Delete). Fetch components are used to get data from the server using a query, to represent it to the user. While action components are used to mutate data on the server. All component declarations run on the client. Like models, UI component names must follow the PascalCase convention.

### Render Expressions

UI structures can be built using the render expression which, takes XML-like structures. The DSL provides a modified version of HTML called XRA that can be mixed with for-expressions, if-expressions, variables, literals, or user-defined UI components. Elements within the render expression can be styled using any CSS classes provided by the WindiCSS[@windi-css-ref], which is a utility-first CSS framework integrated natively in the DSL. Within render expressions, literals and variables can be rendered using the `{ ... }` notation as follows:

\begin{lstlisting}[caption=Rendering literals and variables example.,label={lst:literals-example},language=Thoth]
render(<div>{ "Hello, world" }</div>) // renders a string
render(<div>{ 42069 }</div>) // renders a number
render(<div>{ user.firstName }</div>) // renders a variable
\end{lstlisting}

if-expressions can be declared using the following syntax:

\begin{lstlisting}[caption=If-expression example.,label={lst:if-expr-example},language=Thoth]
// if-expression
render(
  <div>
    [% if condition %]
      { "Condition is true" }
    [% endif %]
  </div>
)

// if-else-expression
render(
  <div>
    [% if condition %]
      { "Condition is true" }
    [% else %]
      { "Condition is false" }
    [% endif %]
  </div>
)
\end{lstlisting}

Moreover, for-expressions can be declared using a similar syntax as if-expression:

\begin{lstlisting}[caption=For-expression example,label={lst:for-expr-example},language=Thoth]
render(
  <div>
    [% for element in list %]
      { element }
    [% endif %]
  </div>
)
\end{lstlisting}

### General Components

Typically, general components are used to build stateless UI components. This is useful to make building UIs more modular and makes maintaining UI components easier. The example below shows a component that takes an argument `post` of type `Post` as an argument and then presents the `title` and `content` with a text of color gray using the CSS classes provided by WindiCSS.

\begin{lstlisting}[caption=General component example.,label={lst:general-component-example},language=Thoth]
component PostDetailComponent(post: Post) {
  render(
    <div className="text-gray-800">
      <div>{ post.title }</div>
      <div>{ post.content }</div>
    <div>
  )
}
\end{lstlisting}

### Fetch Components

There are two types of components that fetch data using a read query. These are `FindUnique` and  `FindMany` components. Fetch components must implement a JSON-like body that takes 4 required entries: `findQuery`, `onError`, `onLoading`, and `onSuccess`. The `findQuery` entry takes a query of type `FindMany` or `FindUnique` depending on the component type. This query is used ot fetch the data from the server. While the entries must take a render expression with a UI structure to view for the user on each case. For example, we can implement a `FindMany` component that presents a list of posts to the user using the query shown in Listing \ref{lst:find-many-query}, as follows:

\begin{lstlisting}[caption=FindMany component example.,label={lst:fetch-example},language=Thoth]
component<FindMany> PublishedPostsList {
  findQuery: getPosts({
    search: {
      published: true
    }
  }) as posts,
  onError: render(<div>{ "Error getting posts" }</div>),
  onLoading: render(<div>{ "Loading posts..." }</div>),
  onSuccess: render(
    <div>
      [% for post in posts %]
        <PostDetailComponent post={ post } />
      [% endfor %]
    </div>
  )
}
\end{lstlisting}

The above example defines a component that uses the `getPosts` query to get a list of published posts and then saves the result in the variable `posts`. When a `FindMany` query is used, it can take an optional JSON object to set values for the search fields defined in the query. In each case, the component will render the appropriate UI structure for the user. As shown in line 12, the component makes use of the component implemented in Listing \ref{lst:general-component-example} to show the post details.

### Action Components

Action components provide a way for developers to create user interfaces that enable users to mutate data by performing actions, such as creating, updating, and deleting. `Create` components, for example, are used to create forms that enable users to add new data to a particular model. These components utilize a query to define the fields that the form should contain. `Update` components, on the other hand, are used to create forms that enable users to update existing data in a model. These components are similar to `Create` components but include pre-populated values for the fields that the user is updating. Finally, `Delete` components are used to create action buttons that enable users to delete a record from a particular model.

#### Create and Update Forms

`Create` and `Update` components has several entries that define their behavior and structure. Firstly, the `actionQuery` entry specifies a query that will be executed when the form is submitted. This query is responsible for creating a new record in the database with the data submitted in the form or updated an existing one. Secondly, the `formInputs` entry is used to define the input fields in the form. Each input field is defined by a name and an input type. Additional entries can also be added to the input field definition, such as styles, placeholders, and default values. The `label` entry can also be used to define a label for each input field, improving the accessibility and usability of the form. Thirdly, the `formButton` entry is used to define the button that will submit the form. This entry allows developers to customize the text displayed on the button, as well as any styles that should be applied to the button. Finally, the `globalStyle` entry is used to define the global styles that will be applied to the entire form. This entry allows developers to customize the appearance of the form, such as the background color and font styles.

The DSL provides a range of input types that developers can use to create type-safe forms for their web applications. The available input types include `TextInput`, `EmailInput`, `PasswordInput`, `NumberInput`, `RelationInput`, `DateTimeInput`, `DateInput`, and `CheckboxInput`. The `TextInput` input type is used to capture textual data, while the `EmailInput` input type is specifically designed to capture email addresses. The `PasswordInput` input type is used to capture passwords, with the input field automatically masking the characters entered. These three input types can be implemented with fields of type `String`. The `NumberInput` input type is used to capture numerical data, such as age or quantity, and can be used with fields of type `Int` only. The `RelationInput` input type is used to capture relational data, such as linking a user to a post they authored. The `DateTimeInput` input type is used to capture both a date and time, while the `DateInput` input type is used to capture only a date. These two input types can be used with `DateTime` fields. Finally, the `CheckboxInput` input type is used to capture `Boolean` data, such as selecting a checkbox to indicate agreement to terms and conditions.

The code shown in Listing \ref{lst:create-form-example} implements a `Create` component, which uses the `createPost` query implemented in Listing \ref{lst:create-query} as its action query. Moreover, the component implements form inputs, which are inputs specified in the `createPost` query. It implements two text inputs for the `title` and `content` fields. Additionally, it implements an input `user` of type `RelationInput`, which specifies that the submitted post should be linked to the currently logged-in user. The value for this input is pre-populated by default and hidden from the user. The form cannot implement fields that are not specified in the `createPost` query. It also must implement all the fields specified by the query. In the end, the component defines a form button, which is the button used to submit the data. The provided example uses the `globalStyle` entry to define global styles for the form elements. In the `globalStyle`, the `form` entry defines the background color of the form, while the `inputContainer` and `input` entries define the background color of the input container and the input field, respectively. The `inputLabel` entry defines the style of the label associated with the input field, and the `inputError` entry defines the style of the error message displayed if the input is invalid. In addition to the global styles, developers can also define styles for individual input fields using the `style` entry. In the example provided, the `style` entry is used to override the global styles to set a different background color and text color to elements in the `title` input field. The styling for this example is represented by the diagram in Figure \ref{fig:form-styling}.

\newpage

\begin{lstlisting}[caption=Create form example.,label={lst:create-form-example},language=Thoth]
component<Create> PostCreateForm {
  actionQuery: createPost(),
  globalStyle: {
    form: "bg-green-500",
    inputContainer: "bg-blue-500",
    input: "bg-red-500",
    inputLabel: "text-black",
    inputError: "text-red-500"
  },
  formInputs: {
    title: {
      style: "bg-white",
      label: {
        style: "text-red-500",
        name: "Post Title"
      },
      input: {
        type: TextInput,
        placeholder: "Enter post title",
        style: "bg-white text-red-500"
      }
    },
    content: {
      label: {
        name: "Post Content",
      },
      input: {
        type: TextInput,
        placeholder: "Enter post content"
      }
    },
    user: {
      input: {
        type: RelationInput,
        isVisible: false,
        defaultValue: connect id with LoggedInUser.id
      }
    }
  },
  formButton: {
    style: "bg-yellow-500",
    name: "Create"
  }
}
\end{lstlisting}

![A diagram representing how form styling works.](source/figures/form-styling.pdf "Form styling"){#fig:form-styling width=500}

#### Delete Buttons

`Delete` buttons are created like `Create` and `Update` components but `Delete` buttons only implement `actionQuery` and `formButton` entries. The example below uses the `deletePostById` query implemented in Listing \ref{lst:delete-query} and passes the value of the argument `id` to the `where` argument of the query.

\newpage

\begin{lstlisting}[caption=Delete button example.,label={lst:delete-button-example},language=Thoth]
component<Delete> PostDeleteButton(id: Int) {
  actionQuery: deletePostById({
    where: id
  }),
  formButton: {
    style: "bg-yellow-500",
    name: "Delete"
  }
}
\end{lstlisting}

## Pages

Pages in the DSL are defined by `page` declarations. Pages are created by composing one or more components and they define the layout and structure of the page. When a user navigates to a specific route, the corresponding page is displayed. Page names must follow the PascalCase convention. Every page must implement an attribute called `@route`, which takes a string to define the route for the page. In the example below, the page `Home` renders the two components `PublishedPostsList` and `PostCreateForm` implemented in Listing \ref{lst:fetch-example} and Listing \ref{lst:create-form-example} consecutively. This page can be accessed by navigating to the root route of the app.

\begin{lstlisting}[caption=Page example.,label={lst:page-example},language=Thoth]
@route("/")
page Home {
  render(
    <PublishedPostsList />
    <PostCreateForm />
  )
}
\end{lstlisting}

## Authentication and Permissions

Authentication is a common feature in modern web apps and it is important to provide developers the ability to implement this feature in their apps in the simplest way possible. To implement authentication in the app, developers have to create a data model that represents users and then define in the app configuration that the app should have authentication using this data model. The user model must define the fields shown in Listing \ref{lst:user-auth-model}. Then to make this model used by the DSL use this model as the user model for the app, developers must implement the following configurations shown in Listing \ref{lst:auth-config} in the app declaration. The entry `userModel` takes a reference to the user model as indicated by the name. While entries in lines 5-9 reference fields in the user model. The entry `onSuccessRedirectTo` is used to define where the user should be redirected to when they login to the app. While the entry `onFailRedirectTo` is used to define where the user should be redirected to when they try to access an app resource that requires authentication without being logged in.

\begin{lstlisting}[caption=User model example.,label={lst:user-auth-model},language=Thoth]
model User {
  id         Int      @id
  username   String   @unique
  password   String
  isOnline	 Boolean  @default(false)
  lastActive DateTime @default(Now)
  ...other fields...
}
\end{lstlisting}

\newpage

\begin{lstlisting}[caption=Authentication configuration example.,label={lst:auth-config},language=Thoth]
app AppWithAuth {
  title: "App with Authentication",
  auth: {
    userModel: User,
    idField: id,
    isOnlineField: isOnline,
    lastActiveField: lastActive,
    usernameField: username,
    passwordField: password,
    onSuccessRedirectTo: "/",
    onFailRedirectTo: "/login"
  }
}
\end{lstlisting}

To create sign up and login forms, developers can do that by creating components of type `SignupForm` or `LoginForm`. The `SignupForm` and `LoginForm` are created like `Create` components. The `SignupForm` must implement all the required fields of the user model. While `LoginForm` must implement the `username` and `password` fields only. The example in Listing \ref{lst:login-form-example} shows how a simple login form can be created. Moreover, `LogoutButton` are created like `Delete` components as shown in Listing \ref{lst:logout-button-example}.

\begin{lstlisting}[caption=Login form example.,label={lst:login-form-example},language=Thoth]
component<LoginForm> MyLoginForm {
  formInputs: {
    username: {
      input: {
        type: TextInput,
        placeholder: "Enter username"
      }
    },
  	password: {
      input: {
        type: PasswordInput,
        placeholder: "Enter password"
      }
    }
  },
  formButton: {
    name: "Login"
  }
}
\end{lstlisting}

\begin{lstlisting}[caption=Logout button example.,label={lst:logout-button-example},language=Thoth]
component<LogoutButton> MyLogoutButton {
  formButton: {
    name: "Logout"
  }
}
\end{lstlisting}

Users might need to grant some permissions before interacting with specific parts of the app. For example, users might need to be authenticated to view a certain page or do some operation. This can be achieved in the DSL by using the `@permissions` attribute and it can take either one of the following values `IsAuth` or `OwnRecord`. The `IsAuth` value makes sure that only authenticated users have permission to view this page, for example. While `OwnsRecord` is used to specify that users can do some operations only on their own records in the database. The permission attribute can be used only be used with `query` and `page` declarations. When it is used with the `query` declaration, it means that only users who have the specified permissions can do the operation. But, when it is used with the `page` declaration, it can only take `IsAuth` as a value and it means that only authenticated users can view the page.

The code in Listing \ref{lst:permissions-example} shows how the permissions attribute can be used with queries and pages by implementing a `FindMany` query which specifies that users can only fetch their own records from the `Post` model when they are authenticated. Furthermore, it implements a page that users can only view when they are authenticated.

\newpage

\begin{lstlisting}[caption=Permissions example.,label={lst:permissions-example},language=Thoth]
@model(Post)
@permissions(IsAuth, OwnsRecord)
query<FindMany> getPostsWithPermissions { /** body */ }

@route("/posts")
@permissions(IsAuth)
page PostsListPage { /** body */ }
\end{lstlisting}

## Customization and Inline TypeScript

The DSL offers developers the option to write inline TypeScript code in the DSL. Also, the DSL allows developers to import JavaScript/TypeScript libraries. This allows developers to write custom code when the DSL becomes limiting. First, to import a JavaScript/TypeScript library developers have to specify the name of the package and the version in the `app` declaration as shown in Listing \ref{lst:app-with-deps-example}.

\begin{lstlisting}[caption=Importing a JavaScript library example.,label={lst:app-with-deps-example},language=Thoth]
app AppWithTSCode {
  title: "App with TS code",
  serverDep: [
    ("is-even", "^1.0.0")
  ],
  clientDep: [
    ...client dependencies...
  ]
}
\end{lstlisting}

The syntax for declaring custom queries and components in our DSL is the same as for default ones but they implement different entries. Both require a JSON-like object that must have a `fn` entry, which takes a TypeScript function as its value. Additionally, an optional entry can be included called `imports` to specify the imports required for the query or component. The TypeScript code must be defined inside the following notation `[| ... |]`.

The query custom function must be a TypeScript ExpressJS arrow function and it is an async function by default. Within the query function, developers can use Prisma using the Prisma client variable `prismaClient`. Prisma is the ORM used on the server to communicate with the database. Furthermore, the return value of all custom queries must be defined in the query declaration signature. If the return value of a query does not match a defined data model, developers can create custom types using a declaration called `type`. The `type` declaration provides a way to create custom types in the DSL to insure type safety of custom queries. All custom queries must have a route that specifies one of the following HTTP methods: get, post, put, and delete to use when calling this query. The custom query shown in Listing \ref{lst:custom-query-example} receives a number from the client, checks if it is even, save the number in a table called `EvenNumber`, then returns `IsEvenMessage` back to the client.

Custom queries can be used with default components based on the HTTP method specified in the `@route` attribute. For instance, queries that implement the `post` method must be used in `Create` components. While queries that implement the `put` method must be used in `Update` components. Queries that implement the `get` method can be used with `FindUnique` and `FindMany` components based on their return value. If the return value is a list then the query can only be used with `FindMany` components, otherwise, it can only be used with `FindUnique` component. Finally, queries that implements the `delete` method must only be used with `Delete` components.

Custom components can take arguments, and be used in other components or pages, and any query defined in the app can be used in the custom components. Queries defined in the app can be accessed using the `Queries` object in the custom TypeScript code. The code in the `fn` entry should take a valid React functional component body code. For example, the code shown in Listing \ref{lst:custom-component-example} implements a custom component that takes the username of the currently logged-in user as an argument and has a form that takes a number and submits it to the `isEven` query implemented in Listing \ref{lst:custom-query-example}. After submitting the form the component will show either "Even" or "Odd" to the user.

\newpage

\begin{lstlisting}[caption=Custom query example.,label={lst:custom-query-example},language=Thoth]
model EvenNumber {
  id     Int @id
  number Int
}

type IsEvenMessage {
  number  Int
  isEvent Boolean
}

@route("post", "/is-even")
query<Custom> isEven : IsEvenMessage {
  imports: [|
    import isEven from "is-even";
  |],
  fn: [|
    (req: Request, res: Response, next: NextFunction) => {
      // get the number from the request body
      const number = req.body.number;
      // check if the number is even using the is-even library
      if (isEven(number)) {
        // save the number in the EvenNumber table
        await prismaClient.evenNumber.create({
          data: { number }
        });
      }

      // send a response back to the client
      res.send({ number, isEven: isEven(number) });
    }
  |]
}
\end{lstlisting}

\newpage

\begin{lstlisting}[caption=Custom component example.,label={lst:custom-component-example},language=Thoth]
component<Custom> IsEvenComponent(username: String) {
  imports: [|
    import React, { setState } from "react";
  |],
  fn: [|
    // declare form data local state
    const [fromData, setFormData] = setState<{ number: number }>({ number: null });
    //declare response data local state
    const [responseData, setResponseData] = setState<{
      number: number;
      isEven: boolean;
    }>({
      number: null,
      isEven: null
    });

    // updates the form data state with the change
    const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
      setFormData({ number: event.target.number.value })
    }

    // submits the form data to the isEven query and updates the component state with the response
    const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
      // the query is used with the fetch API
      const response = await fetch(Queries.isEven(), {
        method: "post"
        body: JSON.stringify({ ...formData })
      });

      if (response.ok) {
        const data = await response.json();
        setResponseData(data);
      }
    }

    return (
      <>
        { responseData.isEven !== null ? (`Number ${responseData.number} is ${responseData.isEven ? "even" : "odd"}, ${username}`) : "" }
        <form onSubmit={handleSubmit}>
          <input
            type="number" name="number"
            value={responseData.number}
            onChange={handleInputChange}
          />
          <input type="submit" value="Submit"/>
        </form>
      </>
    )
  |]
}
\end{lstlisting}
