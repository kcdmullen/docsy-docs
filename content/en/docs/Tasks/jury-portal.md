---
title: 'Jury Service Portal'
date: 2021-10-21
weight: 2
description: >
  Basic info on how the Jury Portal is structured and how it operates.
---

{{% pageinfo %}}
The **Jury Portal** is deployed at [criminaljury.knoxtn.gov](https://criminaljury.knoxcountytn.gov) (prod) and [testcriminaljury.knoxcountytn.gov](https://testcriminaljury.knoxcountytn.gov) (dev).
{{% /pageinfo %}}

The Jury Service Portal is a web application for summoned jurors in Knox County to fill out and submit the forms needed for jury service.

The portal is built with [Laravel]('https://laravel.com/docs/8.x') and [Vue v.2]('https://vuejs.org/'), using the [Vuetify]('https://vuetifyjs.com/en/') Material Design framework.

### Configure and run

Assuming that Laravel, [Composer]('https://getcomposer.org/') and PHP are already installed:

- Run `composer install` (at the project's root directory)
- Run `npm install`
- Set up the `.env` file in the root directory of this project to connect to your database.

### Notable dependencies

- [Vuetify](https://vuetifyjs.com/en/) - material design elements
- [Vuex](https://vuex.vuejs.org/) - state management
- [Vue Router](https://router.vuejs.org/) - client-side routing
- [Axios](https://axios-http.com/) - for http requests
- [Vue-the-mask](https://www.npmjs.com/package/vue-the-mask) for formatting the display of phone numbers

### What the app does

The Juror Service Portal interacts with a **mySQL** database hosted on AWS, and (in the case of the main Juror Info Form), with the Jury Information Management System. When a prospective juror enters their juror ID (which they received in their jury summons) and date of birth, the app checks for a match in a table of jurors ('juror_info', which is part of the mySQL DB). If a match is found, the juror then selects the path for which they were summoned (Grand Jury or Regular), and then steps through a series of required forms.

During the process of stepping through the forms, the app makes these database queries:

- When the **Personal Information Form** loads, the app makes a query to the SQL table **juror-forms** to see which forms, if any, the juror has filled out already. The juror is prevented from filling these out again.
- When the juror chooses to fill out the **Regular Jury Form** or the form to reschedule jury service, the app queries the SQL tables to find open service periods and panels, and populates the drop-down menus with the open dates. On the Regular Jury path, the dates and panels are stored in panels and updated by the court staff via OutSystems. On the Grand Jury path, the service dates are statically stored in the table **grand-jury-dates**, and are occasionally updated by our team at the request of the court staff.

When complete, the juror's information is saved to a 'juror_forms' table in the database. In addition, if the form submitted is the one for regular jury service, the app posts the data via an API call to OutSystems.

If a form fails to submit for reasons other than internet connection issues, the error message and details should be recorded in the table **error_logs**

### Summary of structure

#### Vue

- App instantiated and routes defined - `resources/js/app.js`
- Theme variables - `resources/js/vuetify.js`
- Vue components - `resources/js/components`
- Component on the `/` route - `resources/js/components/StepperComponent.vue`
- Vuex stores
  - `resources/js/store/modules/juror.js` - Methods for retrieving juror data and submitting forms
  - `.../servicePeriods.js` - Methods for retrieving available dates for Grand/Regular Jury service
  - `.../errors.js` - Methods for recording errors to the error log
  - `.../fixedData.js` - For storing and retrieving static data that is used in multiple places
- API calls - `resources/js/api/api.js`
- The components that begin with 'Base' (`Baseinput.vue`, for example), can be used anywhere in this app without being imported (unlike regular components). This is made possible by some setup code in `src/main.js`. The 'Base' components are small, reusuable items like form elements, a progress bar, a confirm dialog, etc. Reusing these instead of making new ones over and over makes it much easier to update the app.

#### Laravel

- Vue app is served through - `resources/views/layout.blade.php`
- Controllers - `app/Http/Controllers` - This is where the database queries are defined
- Models - `app/Models`
- Routes - `routes/web.php`
