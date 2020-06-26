# Keep your Try / Catches as small as possible

This will also help you keep more happy path code to the left.

The goal is to keep just the code you think might throw an exception in the
try/catch. Don't use the "try" clause as a happy path container.

Don't
```
let app;
try {
  await app = createApp();
  new FlashMessage(`Your app is being installed`);
  return app;
} catch (error) {
  showAppInstallationErrorFlashMessage();
  throw error;
}
```

Do:
```
let app;
try {
  await app = createApp();
} catch (error) {
  showAppInstallationErrorFlashMessage(name, clusterId, error);
  throw error;
}

new FlashMessage(`Your app is being installed`);
return app;
```