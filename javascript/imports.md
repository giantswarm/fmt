# Imports

Imports should be sorted alphabetically. That's all.

When the order matters, (for example when importing css files), then this rule
can be ignored.

Example:
```javascript
import 'babel-polyfill';
import adminLogin from './auth/admin';
import configureStore from '../stores/configureStore';
import forgot_password_index from './forgot_password/index';
import forgot_password_set_password from './forgot_password/set_password';
import history from '../stores/history';
import Layout from './layout';
import login from './auth/login';
import logout from './auth/logout';
import oauth_callback from './auth/oauth_callback.js';
import React from 'react';
import signup from './signup/index';
import { ConnectedRouter } from 'connected-react-router';
import { Provider } from 'react-redux';
import { render } from 'react-dom';
import { Route, Switch } from 'react-router-dom';

import 'normalize.css';
import 'bootstrap/dist/css/bootstrap.min.css';
import '../styles/app.sass';
import 'react-datepicker/dist/react-datepicker.css';
```
