# FAQ

<svelte:head><title>FAQ</title></svelte:head>

### I see the data in $form, but it's not posted to the server?

The most common mistake is to forget the `name` attribute on the input field. If you're not using `dataType: 'json'` (see [nested data](/concepts/nested-data)), the form is treated as a normal HTML form, which requires a name attribute for posting the form data.

---

### How can I return additional data together with the form?

You're not limited to just `return { form }` in load functions and form actions, you can return anything else together with the form variables (which can also be called anything you'd like).

```ts
const loginForm = await superValidate(request, loginSchema)
const registerForm = await superValidate(request, registerSchema)
const userName = locals.currentUser.name

return { loginForm, registerForm, userName }
```

If you return this in a load function, it can be accessed in `PageData` in `+page.svelte`. By returning it in a form action, it can be accessed in `ActionData` instead.

---

### What about the other way around, posting additional data to the server?

Conversely, you can add additional form fields not included in the schema, including files (see next question), and also add form data in [onSubmit](/concepts/events#onsubmit), to send extra data to the server. They can then be accessed with `request.formData()` in the form action:

```ts
export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const form = await superValidate(formData, schema);

    if (!form.valid) return fail(400, { form });

    if(formData.has('extra')) {
      // Do something with the extra data
    }

    return { form };
  }
}
```

---

### How to handle file uploads?

File uploads are not handled by Superforms. Fields containing files will be `undefined` in `form.data` after validation, so they need to have names that doesn't conflict with the fields in the schema. 

The recommended way to handle files is to grab the `FormData` and extract the files from there, after validation:

```ts
export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const form = await superValidate(formData, schema);

    if (!form.valid) return fail(400, { form });

    const file = formData.get('file');
    if (file instanceof File) {
      // Do something with the file.
    }

    return { form };
  }
}
```

---

### Can I use endpoints instead of form actions?

Yes, there is a helper function for constructing an `ActionResult` that can be returned from SvelteKit [endpoints](https://kit.svelte.dev/docs/routing#server). See [the API reference](/api#actionresulttype-data-options--status) for more information.

---

### Can a form be factored out into a separate component?

Yes - this question now has its own [article page here](/components).

---

### I want to reuse common options, how to do that easily?

When you start to configure the library to suit your stack, you can create an object with default options that you will refer to instead of `superForm`:

```ts
import type { ZodValidation } from 'sveltekit-superforms';
import { superForm as realSuperForm } from 'sveltekit-superforms/client';
import type { AnyZodObject } from 'zod';

export type Message = { 
  status: 'success' | 'error' | 'warning'; 
  text: string 
};

export function superForm<T extends ZodValidation<AnyZodObject>>(
  ...params: Parameters<typeof realSuperForm<T, Message>>
) {
  return realSuperForm<T, Message>(params[0], {
    // Your defaults here
    errorSelector: '.has-error',
    delayMs: 300,
    ...params[1]
  });
}
```
