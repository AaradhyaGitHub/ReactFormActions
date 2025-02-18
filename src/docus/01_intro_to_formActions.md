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
<form onSubmit={handleSubmit}>
.
.
.
</form>
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
  }

<form action={signupAction}>... </form>
```