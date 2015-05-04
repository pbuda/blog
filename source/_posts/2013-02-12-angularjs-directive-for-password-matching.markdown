---
layout: post
title: "Angular directive for password matching"
date: 2013-02-12 11:30
comments: true
categories: 
---
Recently at [Bootstrap](http://bootstrap.softwaremill.com/#/) I was working on the profile page part. I had to introduce a simple validation of passwords - when you type in your password, you often have to repeat it. This was implemented in a few places already, however it wasn't correct - changing the original password did not trigger validation.

To see how this works, just go to the [Bootstrap page](http://bootstrap.softwaremill.com) and play with the registration form's "password" and "repeat password" inputs. All code snippets are taken from [Bootstrap](http://bootstrap.softwaremill.com/#/).

### Requirements

We have two input fields and we just need to validate whether values in them are synchronized. Values entered in both these fields need to be the same, if they're not, then display error message next to the second input field that the contents are invalid.

### Simple approach

When I started, there was something implemented already. It was a simple solution that based on the [ng-change](http://docs.angularjs.org/api/ng.directive:ngChange) directive. This directive was attached to the repeated field and every time the value changed, method checkPassword() in RegisterController was called.

``` html How checkPassword() was used
<input type="password" name="repeatPassword" id="repeatPassword" placeholder="repeat password" ng-model="user.repeatPassword" ng-change="checkPassword()" required>
```

``` javascript Validation in checkPassword method
$scope.checkPassword = function () {
    $scope.registerForm.repeatPassword.$error.dontMatch = $scope.user.password !== $scope.user.repeatPassword;
};
```
The dontMatch error flag was then used to display error message.
``` html Using dontMatch flag to display error
<span class="text-error" ng-show="registerForm.repeatPassword.$dirty && registerForm.repeatPassword.$error.dontMatch">Passwords don't match!</span>
```

This isn't the best solution. First of all, this only checked equity upon changing the repeated field, not the original password field. To make this work it would be required in both original and repeated fields. This also wasn't reusable - to use it in another place, checkPassword method had to be actually copied.

### Angular directive to the rescue

Directives in AngularJS are used to create reusable components so I thought about how to use them to solve the validation problem. It was rather easy and straightforward.

``` javascript The repeatPassword directive
directives.directive("repeatPassword", function() {
    return {
        require: "ngModel",
        link: function(scope, elem, attrs, ctrl) {
            var otherInput = elem.inheritedData("$formController")[attrs.repeatPassword];

            ctrl.$parsers.push(function(value) {
                if(value === otherInput.$viewValue) {
                    ctrl.$setValidity("repeat", true);
                    return value;
                }
                ctrl.$setValidity("repeat", false);
            });

            otherInput.$parsers.push(function(value) {
                ctrl.$setValidity("repeat", value === ctrl.$viewValue);
                return value;
            });
        }
    };
});
```
``` html Using repeatPassword directive
<input type="password" name="repeatPassword" id="repeatPassword" placeholder="repeat password" ng-model="user.repeatPassword" repeat-password="password" required>
```

At first I was trying to attach $parser function to just the element the directive was attached to. But this didn't work, because when I was comparing to the value stored in controller the results not always were correct. The reason for this is that invalid values are not passed to controller, and so when one value was invalid, it wasn't present in controller, and directive failed. As you can see, the trick is to attach $parsers to each control that requires syncing. This took me some time to figure out, but for now it works quite nice.

### Summary

Angular directives are quite nice. They can be used to create reusable components and in this case, I created something that was automatically usable in three places, without the need to writing any controller code. Moreover, this directive is not only for passwords - rename that to stringsMatch and you get a nice directive that checks if you string values are in sync. Neat!

This change has been introduced in [Bootstrap](http://bootstrap.softwaremill.com/#/) in [this commit](https://github.com/softwaremill/bootstrap/commit/515d289ddea2159b8c3eaa956cdfb658898b5358).