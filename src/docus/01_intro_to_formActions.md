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

# **Intro to useActionState Hook**

Now, we have an `errors` array in the action form that we need to access somehow.

## **Understanding `useActionState`**

The `useActionState` hook is designed to work with form actions. It helps us manage the form’s state, handle errors, and track whether the form is currently submitting.

### **First Parameter: The Action Function**

The first argument we pass to `useActionState` is our action function, which handles form submission.

```jsx
useActionState(signupAction);
```

By providing this function, React can enhance it behind the scenes, allowing us to better manage the form’s state.

### **Second Parameter: The Initial State**

The second argument is the initial state object for the form:

```jsx
useActionState(signupAction, { errors: null });
```

This is similar to how `useState` works—this object holds our form's initial state, such as error messages.

### **What Does `useActionState` Return?**

Unlike `useState`, which returns two values, `useActionState` returns an **array with three elements**:

1. **Current Form State**
   - Initially, this is the **initial state** (the second argument passed to `useActionState`).
   - Once the action function executes, this state gets updated with the returned value from the action.

2. **Updated Form Action**
   - `useActionState` wraps the provided action function and creates an enhanced version of it.
   - This is necessary because React ensures that form state updates correctly through this function.
   - We pass this enhanced function to the `action` attribute of the form.

3. **Pending State**
   - A boolean (`true` or `false`) indicating if the form is currently submitting.
   - Useful when dealing with asynchronous operations (e.g., showing a loading indicator while submitting).

## **Using `useActionState` in Practice**

To set up the hook:

```jsx
const [formState, formAction] = useActionState(signupAction, { errors: null });
```

Here:
- `formState` holds the current form state (starting with `{ errors: null }`).
- `formAction` is the enhanced version of `signupAction` that we use in our `<form>`.

## **Updating the Action Function**

When using `useActionState`, we need to modify our original action function to accept **two parameters**:

```jsx
function signupAction(prevFormState, formData) {
  // Handle form submission logic here
}
```

### **Why Do We Need Two Parameters?**

Using just the `formData` object is not enough because we also need access to the previous form state. This allows us to:
- Retain and update error messages.
- Maintain additional form-related data.
- Ensure the form state is correctly updated after submission.

## **Using `formState` in JSX**

Now, we can extract information from `formState` and display error messages dynamically:

```jsx
{
  formState.errors && (
    <ul>
      {formState.errors.map(error => (
        <li key={error}>{error}</li>
      ))}
    </ul>
  )
}
```

