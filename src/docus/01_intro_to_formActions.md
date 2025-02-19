# **Intro to Form Actions**

Available in React v.19 or higher.

## **Traditional Form Submission**

Nothing wrong with this approach:

```jsx
function handleSubmit(event) {
  event.preventDefault();
}
```

And we bind the event listener to the component:

```jsx
<form onSubmit={handleSubmit}>. . .</form>
```

## **Using the `action` Prop**

We can also use the newer `action` prop instead of `onSubmit`.

Even in non-React projects, we can set the `action` attribute. This is usually the path to where the form will be sent.

In React, it's different. React overwrites the attribute and ensures that the function executes when the form is submitted. Behind the scenes, it will call `preventDefault` and execute our custom function.

Instead of an event object, when using `action`, we get a `formData` object.

`formData` then contains all the inputted information, as long as the `name` prop is included in the input components.

## **Implementing Form Action**

```jsx
function signupAction(formData) {
  const email = formData.get("email");
  const password = formData.get("password");
  const confirmPassword = formData.get("confirm-password");
  const firstName = formData.get("first-name");
  const lastName = formData.get("last-name");
  const role = formData.get("role");
  const terms = formData.get("terms");
  const acquisitionChannel = formData.getAll("acquisition");

  let errors = [];

  if (!isEmail(email)) {
    errors.push("Invalid email address");
  }
  if (!isNotEmpty(password) || !hasMinLength(password, 6)) {
    errors.push("Please provide a password with at least six characters");
  }
  if (!isEqualToOtherValue(password, confirmPassword)) {
    errors.push("Passwords do not match ");
  }
  if (!isNotEmpty(firstName) || !isNotEmpty(lastName)) {
    errors.push("Please provide both your first and last name ");
  }
  if (!isNotEmpty(role)) {
    errors.push("Please select a role");
  }
  if (!terms) {
    errors.push("You must agree to the Terms and Conditions");
  }
  if (acquisitionChannel.length === 0) {
    errors.push("Please select at least one acquisition channel");
  }

  if (errors.length > 0) {
    return {
      errors
    };
  }

  return { errors: null };
}

<form action={signupAction}>... </form>;
```

---

# **Intro to useActionState hook**

Now, we have an errors array in the action form that we need to access somehow.
useActionState hook wants an action function as a first parameter:

```jsx
useActionState(signupAction);
```

This hook works with form actions. The second parameter is an initial state value:

```jsx
useActionState(signupAction, { errors: null });
```

Like `useState`, `useActionState` returns an array. But it returns 3 elements:

1. **Current form state** -> Initially the initial state (2nd argument), then the state returned by the action.
2. **Updated formAction** -> Our form action with extra enhancement:
   - We pass the action function as the first argument to `useActionState`.
   - React creates a new function to wrap around this function.
   - This is the `formAction` that we then later pass to `action` on the form element.
3. **Pending** -> `true` or `false`
   - Depending on whether the form is currently being submitted or not.
   - More useful when we deal with asynchronous programming.

### **Setting up the hook:**

```jsx
const [formState, formAction] = useActionState(signupAction, { errors: null });
```

We also now need to change our original action function to accept 2 parameters:

```jsx
function signupAction(prevFormState, formData) {...}
```

But now, we can extract info from this hook and use it in our JSX:

```jsx
{
  formState.errors.map(
    error => (
      <li key={error}>
        {error}
      </li>
    )
  )
}
```

---

## **Preventing Form Reset on Submission**

Currently, when the signup button is clicked, the form submits and resets the input fields, clearing user input even if there are validation errors. To prevent this, we can modify the action function to retain user-entered values when errors are present:

```jsx
if (errors.length > 0) {
  return {
    errors,
    enteredValues: {
      email,
      password,
      confirmPassword,
      firstName,
      lastName,
      role,
      acquisitionChannel,
      terms
    }
  };
}
```

### **How This Works:**
- Instead of returning only the errors, we also return an `enteredValues` object containing the current values of the input fields.
- This ensures that when validation fails, the input values are preserved in the form state.

### **Binding Default Values in JSX:**
We can now use the stored `enteredValues` from `formState` to pre-fill the input fields:

```jsx
defaultValue={formState.enteredValues?.email}
```

For checkboxes and radio buttons, we use `defaultChecked`:

```jsx
defaultChecked={formState.enteredValues?.acquisitionChannel.includes("google")}
defaultChecked={formState.enteredValues?.terms}
```

### **How It All Ties Together:**
1. **The action function** processes the form submission.
   - If there are errors, it returns `errors` along with `enteredValues`.
   - If there are no errors, the form processes normally.
2. **The `useActionState` hook** maintains the form state.
   - It initializes with `errors: null`.
   - If an error occurs, it updates the state to include `enteredValues`.
3. **The form inputs use `defaultValue` and `defaultChecked`** to persist user input.
   - This prevents fields from resetting when submission fails.

This approach improves user experience by ensuring that users don’t lose their input if they make a mistake, allowing them to correct errors without re-entering all their information.

