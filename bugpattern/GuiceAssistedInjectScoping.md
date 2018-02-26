---
title: GuiceAssistedInjectScoping
summary: Scope annotation on implementation class of AssistedInject factory is not allowed
layout: bugpattern
tags: ''
severity: ERROR
providesFix: REQUIRES_HUMAN_ATTENTION
---

<!--
*** AUTO-GENERATED, DO NOT MODIFY ***
To make changes, edit the @BugPattern annotation or the explanation in docs/bugpattern.
-->

## The problem
Classes that AssistedInject factories create may not be annotated with scope annotations, such as @Singleton.  This will cause a Guice error at runtime.

See [https://code.google.com/p/google-guice/issues/detail?id=742 this bug report] for details.

## Suppression
Suppress false positives by adding the suppression annotation `@SuppressWarnings("GuiceAssistedInjectScoping")` to the enclosing element.

----------

### Positive examples
__AssistedInjectScopingPositiveCases.java__

{% highlight java %}
/*
 * Copyright 2013 The Error Prone Authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.google.errorprone.bugpatterns.inject.guice.testdata;

import com.google.inject.Inject;
import com.google.inject.Singleton;
import com.google.inject.assistedinject.Assisted;
import com.google.inject.assistedinject.AssistedInject;
import com.google.inject.servlet.RequestScoped;

/** @author eaftan@google.com (Eddie Aftandilian) */
public class AssistedInjectScopingPositiveCases {

  // BUG: Diagnostic contains: remove this line
  @Singleton
  public class TestClass {
    @Inject
    public TestClass(String unassisted, @Assisted String assisted) {}
  }

  // BUG: Diagnostic contains: remove this line
  @RequestScoped
  public class TestClass2 {
    @Inject
    public TestClass2(String unassisted, @Assisted String assisted) {}
  }

  // BUG: Diagnostic contains: remove this line
  @Singleton
  public class TestClass3 {
    @AssistedInject
    public TestClass3(String param) {}
  }

  /** Multiple constructors, but only one with @Inject, and that one matches. */
  // BUG: Diagnostic contains: remove this line
  @Singleton
  public class TestClass4 {
    @Inject
    public TestClass4(String unassisted, @Assisted String assisted) {}

    public TestClass4(String unassisted, int i) {}

    public TestClass4(int i, String unassisted) {}
  }

  /** Multiple constructors, none with @Inject, one matches. */
  // BUG: Diagnostic contains: remove this line
  @Singleton
  public class TestClass5 {
    public TestClass5(String unassisted1, String unassisted2) {}

    public TestClass5(String unassisted, int i) {}

    @AssistedInject
    public TestClass5(int i, String unassisted) {}
  }

  /** JSR330 annotations. */
  // BUG: Diagnostic contains: remove this line
  @javax.inject.Singleton
  public class TestClass6 {
    @javax.inject.Inject
    public TestClass6(String unassisted, @Assisted String assisted) {}
  }
}
{% endhighlight %}

### Negative examples
__AssistedInjectScopingNegativeCases.java__

{% highlight java %}
/*
 * Copyright 2013 The Error Prone Authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.google.errorprone.bugpatterns.inject.guice.testdata;

import com.google.inject.Inject;
import com.google.inject.Singleton;
import com.google.inject.assistedinject.Assisted;
import com.google.inject.assistedinject.AssistedInject;

/** @author eaftan@google.com (Eddie Aftandilian) */
public class AssistedInjectScopingNegativeCases {

  /** Class is not assisted and has no scoping annotation. */
  public class TestClass1 {
    public TestClass1(String unassisted1, String unassisted2) {}
  }

  /** Class is not assisted and has no scoping annotation, but has an unrelated annotation. */
  @SuppressWarnings("foo")
  public class TestClass2 {
    public TestClass2(String unassisted, @Assisted String assisted) {}
  }

  /** Class is not assisted but has scoping annotation. */
  @Singleton
  public class TestClass3 {
    public TestClass3(String unassisted1, String unassisted2) {}
  }

  /** Class is assisted via @Assisted param but has no scoping annotation. */
  public class TestClass4 {
    @Inject
    public TestClass4(@Assisted String assisted) {}
  }

  /** Class is assisted via @AssistedInject constructor but has no scoping annotation. */
  public class TestClass5 {
    @AssistedInject
    public TestClass5(String unassisted) {}
  }

  /** Class is not assisted -- constructor with @Assisted param does not have @Inject. */
  @Singleton
  public class TestClass6 {
    public TestClass6(@Assisted String assisted) {}
  }

  /** Multiple constructors but not assisted. */
  @Singleton
  public class TestClass7 {
    public TestClass7(String unassisted1, String unassisted2) {}

    public TestClass7(String unassisted, int i) {}

    public TestClass7(int i, String unassisted) {}
  }

  /** Multiple constructors, one with @Inject, non-@Inject ones match. */
  @Singleton
  public class TestClass8 {
    @Inject
    public TestClass8(String unassisted1, String unassisted2) {}

    @AssistedInject
    public TestClass8(String param, int i) {}

    @AssistedInject
    public TestClass8(int i, String param) {}
  }

  /** Multiple constructors, one with @Inject, non-@Inject ones match. */
  @Singleton
  public class TestClass9 {
    @Inject
    public TestClass9(String unassisted1, String unassisted2) {}

    @AssistedInject
    public TestClass9(String param, int i) {}

    @AssistedInject
    public TestClass9(int i, String param) {}
  }

  @Singleton
  public class TestClass10 {
    public TestClass10(@Assisted String assisted, String unassisted) {}

    public TestClass10(@Assisted String assisted, int i) {}

    public TestClass10(int i, @Assisted String assisted) {}
  }
}
{% endhighlight %}
