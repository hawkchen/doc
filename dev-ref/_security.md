
Keikai engine can run in a secure mode to prevent unauthorized connection. That means each browser should connect to keikai engine through java client (your web application)

# How to enable secure mode:

## Run Keikai Engine in secure mode

`./keikai --secureMode`

## Java Client returns a Secure URI
```java
Settings settings = Settings.DEFAULT_SECURE_SETTINGS.clone(); // use the secure settings
// optional: use session id as token
settings.set(Settings.Key.SECURE_TOKEN, ((HttpSession) Executions.getCurrent().getSession().getNativeSession()).getId());
// optional: whether to check request origin header
settings.set(Settings.Key.SECURE_ORIGIN, "*//localhost*");
//connect with above settings
spreadsheet = Keikai.newClient(KeikaiUtil.getKeikaiServerAddress(Executions.getCurrent()), settings);

//pass target element's id and get keikai script URI
String scriptUri = spreadsheet.getSecureURI((HttpServletRequest)Executions.getCurrent().getNativeRequest(),
        (HttpServletResponse) Executions.getCurrent().getNativeResponse(),
        getSelf().getFellow("myss").getUuid());

```