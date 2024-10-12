---
layout: post
title: "Migrating from Hanami 1.3 to Rails 7"
description: "When migrating makes more sense than upgrading."
date: 2024-10-12
category: Tech
tags: [ruby, rails, hanami]
---

In 2017, I created a URL shortener app using Hanami because I like how Hanami tried to separate its web and API layers into separate apps.

In 2024, I tried to upgrade and this happened.

<!--more-->

## Why Hanami?

Simply because the grass feels greener at that time. I was looking at a candidate we interviewed and they were the only candidate using Hanami 1.x. To me, it felt super clean compared to Rails 5.x at that time.

Since then I studied Hanami. I like Hanami's philosophy, where every project is created as API first, web second. Hanami tried to breakdown each of your services inside `apps`. You can read more [here](https://guides.hanamirb.org/v1.3/introduction/getting-started/#hanamis-architecture).

That's when I decided to create my project for URL shortener using Hanami.

## What Happened Next?

After 2019, I don't really code in Ruby anymore. My last exposure to Ruby is with Ruby 2.6.6, which has been deprecated since March 2022. I also got this itch to update my URL shortener, which is turning 7 years old to see how easy it is to maintain. So, I installed Ruby 3.3.5 and starts hacking away.

Well, not really. I got my first roadblock with Hanami.

Hanami 2.0 started with a very different approach than Hanami 1.3. It no longer has `apps` where we can place different modules. It started to look like Rails to me with a single `app`.

At first, I tried to upgrade it using several guides I found online. At one point in time, I also started to create a fresh Hanami project. However, I don't feel as excited as I was before. Probably because the structure reminded me too much of Rails.

Then it got me thinking, what happened to Rails now?

## Why Rails Now?

My last exposure with Rails was Rails 5.1, so I'm taking this opportunity to create a fresh Rails project and I really like it.

I discovered all of these amazing little details in fresh Rails project, such as:
1. Rails is now shipped with Github workflows out of the box.
2. Rails is now shipped with brakeman (for scanning security vulnerabilities).
3. Rails is now shipped with Turbo Rails, which makes HTML pages feels as fast as SPA. This also enabled support for custom alert and dialog out of the box.

When I see the github workflow, I decided to convert the project to Rails. To my delight, the github workflow has most of the basic CI needs covered:
1. It has Brakeman (for scanning security vulnerabilities for Ruby).
2. It has Importmap (for scanning security vulnerabilities for Javascript dependencies).
3. It has Rubocop (for style linter).
4. It runs the test.

Rails 7.2.1 is also shipped with Selenium and Capybara, where you can run system test on your application.

## Lesson Learned

In my personal opinion, I think it's always better to stick with a framework which has been established and has bigger community. On top of that, Rails has excellent convention which hasn't deviate much ever since its inception.

So for now, I will stick with Rails for my project. Time will tell if I will have a change of mind in probably the next 7 years.
