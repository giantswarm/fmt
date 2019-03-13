# Imports

Imports should be sorted by these rules:

1. sort by type: all, many, single
2. then sort alphabetically (not case sensitive)

When the order matters, for example when importing CSS files, then this rule
can be ignored. Blank lines have to be applied to satisfy ESlint.

Example:

```javascript
import * as clusterActions from '../../actions/clusterActions';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import { FlashMessage, messageTTL, messageType } from '../../lib/flash_message';
import _ from 'underscore';
import Button from '../shared/button';
import PropTypes from 'prop-types';
import React from 'react';

// CSS
import 'normalize.css';
import 'bootstrap/dist/css/bootstrap.min.css';

import '../styles/app.sass';
import 'react-datepicker/dist/react-datepicker.css';
```
