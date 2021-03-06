# Validators

## Built in validator functions

The library makes a number of validation factory functions available out of the box that should help with common use cases.

```typescript
import {validators} from '@shopify/react-form-state';
```

```typescript
  // input has a length > the given number
  lengthMoreThan(length: number, errorContent: ErrorContent): Validator
  // input has a length < the given number
  lengthLessThan(length: number, errorContent: ErrorContent): Validator
  // input is a numeric string
  numericString(errorContent: ErrorContent): Validator
  // input must be a non-empty string
  requiredString(errorContent: ErrorContent): Validator
  // input cannot be null/undefined
  required(errorContent: ErrorContent): Validator
```

We can pass these directly into the `validators` prop of `<FormState />`.

```typescript
import {TextField} from '@shopify/polaris';
import FormState, {validators} from '@shopify/react-form-state';

function MyComponent() {
  return (
    <FormState
      initialValues={{
        title: 'Cool title',
        description: 'Cool product',
        quantity: 0,
      }}
      validators={{
        title: validators.lengthLessThan(10, 'That title is too long')

        quantity: [
          validators.required('Products must have a quantity'),
          validators.numericString('Quantity must be numeric'),
        ],
      }}
    >
      {formDetails => {
        const {fields} = formDetails;

        return (
          <form>
            <TextField label="Title" {...fields.title} />
            <TextField label="Description" {...fields.description} />
            <TextField label="Quantity" {...fields.description} />
          </form>
        );
      }}
    </FormState>
  );
}
```

## Building reusable validators

As the number of custom validators in your app grows, you'll likely want to reuse them. This can be tricky because you'll usually want to support custom error messages depending on context.

`@shopify/react-form-state` ships with a small DSL for building your own validators.

```typescript
import {validate} from '@shopify/react-form-state';
```

the `validate` function takes a `Matcher` function and some error content, and returns a `Validator` that works with `FormState`'s `validators` prop.

```typescript
export function validate<Input>(
  matcher: Matcher<Input>,
  errorContent: ErrorContent,
): (input: Input) => ErrorContent | void;
```

You can use `validate` to generate handlers cleanly from your own functions.

```typescript
//utilities.ts
export function isValidEmail(input: string) {
  // example only
  return /+\@.+\..+/.test(string);
}
```

```typescript
//MyPage.ts
import FormState from '@shopify/react-form-state';
import {isValidEmail} from './utilities';

function MyComponent() {
  return (
    <FormState
      initialValues={email: ''}
      validators={{
        email: validate(isValidEmail, 'invalid email')
      }}
    >
      {/* ...some ui */}
    </FormState>
  );
}
```

Or take it a step further by making a generic shared validator.

```typescript
//custom-validators.ts
import {validate, ErrorContent} from '@shopify/react-form-state';

export function validEmail(input: string, errorContent: ErrorContent) {
  // example only
  return validate(input => /+\@.+\..+/.test(input), errorContent);
}
```

```typescript
// MyPage.ts
import FormState from '@shopify/react-form-state';
import {validEmail} from './custom-validators';

function MyComponent() {
  return (
    <FormState
      initialValues={email: ''}
      validators={{
        email: validEmail('Email is invalid'),
      }}
    >
      {/* ...some ui */}
    </FormState>
  );
}
```

`validate`-generated functions also support taking an error-generating functions as their last parameter. Suppose we want to include the current field value in our error message; we could update the above example to use this technique.

```typescript
// MyPage.ts
import {validEmail} from './custom-validators';

function MyComponent() {
  return (
    <FormState
      initialValues={email: ''}
      validators={{
        email: validEmail((input) => `${input} is not a valid email`),
      }}
    >
      {/* ...some ui */}
    </FormState>
  );
}
```

## Compound validators

All of the validation we've done so far has been concerned with scalar value fields. Its possible to write validators for arrays and objects using the syntax described above, but it would be difficult to handle passing errors down intelligently. Its great for things like checking that there are a set number of items in the array, but terrible for checking individual subfield values.

```typescript
validators={{
  variants(input) {
    if (input.length > 10) {
      return 'too many variants!';
    }
  }
}}
```

Because of these difficulties, this package also contains some helpers for writing validators for complex fields that work with `<FormState.Nested />` and `<FormState.List />` to let us propagate errors down the same way as we do for regular fields.

### `validateNested()`

```typescript
import {validateNested} form '@shopify/react-form-state';
```

The validation helper companion for `<FormState.Nested />` lets you define field level validations for nested fields using the same API as you would use for flat scalar fields.

```typescript
  validators={{
    firstVariant: validateNested({
      price: validators.numericString('variant price should be a number'),
      sku: validators.lengthLessthan(3, 'variant SKU must be shorter than 3 characters');
    });
  }}
```

### `validateList()`

```typescript
import {validateList} form '@shopify/react-form-state';
```

```typescript
  validators={{
    variants: validateList({
      price: validators.numericString('variant price should be a number'),
      sku: validators.lengthLessthan(3, 'variant SKU must be shorter than 3 characters');
    });
  }}
```
