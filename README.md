Validity
========

Effortless JavaScript form validation and error handling

### Basic Usage

Include the required scripts in your HTML

```html
<script type="text/javascript" src="validator.min.js"></script>
<script type="text/javascript" src="validity.min.js"></script>
```

#### Client-Side Validation

Simply add a `data-validation` attribute to each input field you would like validated. Set each attribute to a JSON-string dictionary, with each key being the validation type you would like performed on the input and the value being the error message you would like to appear on that input if its corresponding validation fails:

```html
<form id='loginForm'>
  <input type='email' name='email' data-validation='{"required": "Please enter your email", "isEmail": "Please enter a valid email address"}' />
  <input type='password' name='password' data-validation='{"required": "Please enter a password"}' />
  <input type='button' value='Submit' onclick='login()' />
</form>
```

Validity currently supports the following validation types:

- **required** - ensure the field is not empty.
- **isEmail** - ensure the field is a valid email address.
- **isURL** - ensure the field is a valid URL.

Use JavaScript to validate the form before submission. Validation is performed by calling Validity's `validate` function and passing in the id of the form you would like to validate. This function returns `true` if all validations pass and `false` if any of them fail:

```javascript
function login () {
  if (Validity.validate('loginForm')) {
    // post the form data
  } else {
    console.error("There are errors on the form");
  }
}
```

#### Validity helps display errors reported by ther server too

Add a `data-error` attribute to each input field you would like server-side errors to be shown on. Set each attribute to a JSON-string dictionary, with each key being the error returned by the server and the value being the error message you would like to appear on that input if the server returns that error:

```html
<form id='loginForm'>
  <input type='email' name='email' data-error='{"101": "Invalid username or password"}' data-validation='{"required": "Please enter your email", "isEmail": "Please enter a valid email address"}' />
  <input type='password' name='password' data-validation='{"required": "Please enter a password"}' />
  <input type='button' value='Submit' onclick='login()' />
</form>
```

When you receive response from your server, simply call Validity's `displayServerErrors` function, passing in the id of the form you would like to target, along with an array containing the error codes returned by the server.

```javascript
function login () {
  if (Validity.validate('loginForm')) {
    $.post('/login', $('#loginForm').serialize()).then(function (response) {
      if (response.success) {
         // We successfully logged in
         window.location.href = '/loggedin';
      } else {
        // Login failed
        Validity.displayServerErrors('loginForm', [response.error.code.toString()]);
      }
    });
  } else {
    console.error("There are errors on the form");
  }
}
```

### Using Custom Error-handling Functions

The default error-handling function used by `validate` and `displayServerErrors` is called ` two things:

1. Adds a "has-error" class to the parent of the input (assumed to be a "form-group" or similar container for the input and its associated elements)
2. Sets the text of the first label in the form-group to the error message

This is designed to work seamlessly with forms built using Twitter Bootstrap. If you would like to use a custom error-handling function, it is very easy to do so. Let's say that instead of using the default error-handler, we would like to add the "has-error" class to the input element itself, and make the error message appear in the input's placeholder. Let's define a function that does this:

```javascript
function customSetError (element, message) {
  element.classList.add('has-error');
  element.setAttribute('placeholder', message);
}
```

Let's define a function that will clear this error state as well:

```javascript
function customClearError (element) {
  element.classList.remove('has-error');
  element.setAttribute('placeholder', '');
}
```

Now we can use these by passing them into `validateWithCustomFunctions` and `displayServerErrorsWithCustomFunctions`:

```javascript
function login () {
  if (Validity.validateWithCustomFunctions('loginForm', customSetError, customClearError)) {
    $.post('/login', $('#loginForm').serialize()).then(function (response) {
      if (response.success) {
         // We successfully logged in
         window.location.href = '/loggedin';
      } else {
        // Login failed
        Validity.displayServerErrors('loginForm', [response.error.code.toString()], customSetError, customClearError);
      }
    });
  } else {
    console.error("There are errors on the form");
  }
}
```