---
title: "Customize Claims for OpenID Connect"
categories:
  - web-development
  - drupal
tags:
  - OAuth2
  - Decoupled Drupal
  - Drupal Development
image: /assets/images/2020/identity.jpg
---
<p>Drupal, out of the box, cannot provide all the data suggested in the <a href="https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims">OpenID Connect Standard Claims</a>. This means that in order to implement the different claims sites will need to let Simple OAuth know about those implementation details.</p>
<p>In order to add or alter custom claims to the OpenID Connect (aka OIDC) there are two steps necessary:</p>

<!-- more -->


<ul>
	<li>Implement <code class="language-php">hook_simple_oauth_oidc_claims_alter</code> to alter the associative array containing the claims and their values.</li>
	<li>Add the claim names to the <code class="language-php">services.yml</code> listing all the claims available on the responses. Any modules helpfully implementing this hook in their code will need to be whitelisted, so the site is always aware of what claims are being exposed.</li>
</ul>

<h2>Example: Adding the <code class="language-php">phone_number</code> claim</h2>

<em>You can download the code for the <code>simple_oauth_companion</code> module from <a href="https://mateuaguilo.com/assets/documents/simple_oauth_companion.tar.gz">here</a>. Bear in mind that you will still need to edit your <code>services.yml</code> file.</em>

<p>This example assumes that you have added a phone field to the user entity <code class="language-php">field_phone_number</code>. It also assumes that you created a custom module <code class="language-php">simple_oauth_companion</code> to hold the custom code.</p>

<h3>Step 1: simple_oauth_companion.module</h3>

<pre>
<code class="language-php">/**
 * Implements hook_simple_oauth_oidc_claims_alter().
 */
function simple_oauth_companion_simple_oauth_oidc_claims_alter(array &amp;$claim_values, array &amp;$context) {
  $account = $context['account'];
  assert($account instanceof UserInterface);
  $value = $account-&gt;get('field_phone_number')-&gt;getValue();
  $claim_values['phone_number'] = $value[0]['value'] ?? NULL;
}
</code></pre>

<h3>Step 2: web/sites/default/services.yml</h3>

<pre>
<code class="language-php language-yaml"># ... append at the end.
  simple_oauth.openid.claims:
    - sub
    - name
    - preferred_username
    - email
    - email_verified
    - locale
    - profile
    - updated_at
    - zoneinfo
    - phone_number # &lt;-- This is the new claim.
</code></pre>

<h3>Final result</h3>

<p>If all went well the OIDC claims will now contain the additional <code class="language-php">phone_number</code> claim, like below. Check the <code class="language-php">/oauth/userinfo</code> endpoint to see if the data is available.</p>

<pre>
<code class="language-javascript language-php">{
  "sub": "36",
  "name": "test",
  "preferred_username": "test",
  "email": "test1@example.org",
  "email_verified": true,
  "profile": "http:\/\/local.contrib.com\/en\/user\/36",
  "locale": "en",
  "zoneinfo": "Europe\/Madrid",
  "updated_at": "1601238333",
  "phone_number": "+1 985 43 99 01"
}</code></pre>

<small>Photo by <a href="https://unsplash.com/@ko_pth?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Kevin Jesus Horacio</a> on <a href="https://unsplash.com/s/photos/identity?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></small>
