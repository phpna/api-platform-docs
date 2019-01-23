# 测试和指定API

可以在API Platform发行版中轻松安装一组用于指定和测试API的有用工具：

* [PHPUnit](https://phpunit.de/)允许通过单元测试覆盖您的类，并通过Symfony的集成编写功能测试。
* [Behat](http://docs.behat.org/) （一个[行为驱动的开发框架](http://en.wikipedia.org/wiki/Behavior-driven_development)）及其[Behatch扩展](https://github.com/Behatch/contexts) （一组专用于REST API和JSON文档的上下文）可以方便地指定和测试您的API：将API规范编写为用户故事并以自然语言编写然后执行这些方案断言应用程序来验证其行为。

查看[Symfony文档关于测试的部分](https://symfony.com/doc/current/testing.html)，了解如何在API Platform项目中使用PHPUnit。

按照以下步骤安装Behat很容易：

    $ docker-compose exec php composer require --dev behat/behat
    $ docker-compose exec php vendor/bin/behat -V
    $ docker-compose exec php vendor/bin/behat --init

这将在您的项目中安装Behat并创建一个目录 `features`，您可以在其中放置feature文件。

以下是[Gherkin](http://docs.behat.org/en/latest/user_guide/gherkin.html) feature文件的示例，该文件指定了我们在[本教程中创建的bookstore API](index.md)的行为。感谢Behatch，可以针对API执行此feature文件，而无需编写单行PHP。

```gherkin
# features/books.feature
Feature: Manage books and their reviews
  In order to manage books and their reviews
  As a client software developer
  I need to be able to retrieve, create, update and delete them through the API.

  # the "@createSchema" annotation provided by API Platform creates a temporary SQLite database for testing the API
  @createSchema
  Scenario: Create a book
    When I add "Content-Type" header equal to "application/ld+json"
    And I add "Accept" header equal to "application/ld+json"
    And I send a "POST" request to "/books" with body:
    """
    {
      "isbn": "9781782164104",
      "title": "Persistence in PHP with the Doctrine ORM",
      "description": "This book is designed for PHP developers and architects who want to modernize their skills through better understanding of Persistence and ORM.",
      "author": "Kévin Dunglas",
      "publicationDate": "2013-12-01"
    }
    """
    Then the response status code should be 201
    And the response should be in JSON
    And the header "Content-Type" should be equal to "application/ld+json; charset=utf-8"
    And the JSON should be equal to:
    """
    {
      "@context": "/contexts/Book",
      "@id": "/books/1",
      "@type": "Book",
      "id": 1,
      "isbn": "9781782164104",
      "title": "Persistence in PHP with the Doctrine ORM",
      "description": "This book is designed for PHP developers and architects who want to modernize their skills through better understanding of Persistence and ORM.",
      "author": "K\u00e9vin Dunglas",
      "publicationDate": "2013-12-01T00:00:00+00:00",
      "reviews": []
    }
    """

  Scenario: Retrieve the book list
    When I add "Accept" header equal to "application/ld+json"
    And I send a "GET" request to "/books"
    Then the response status code should be 200
    And the response should be in JSON
    And the header "Content-Type" should be equal to "application/ld+json; charset=utf-8"
    And the JSON should be equal to:
    """
    {
      "@context": "/contexts/Book",
      "@id": "/books",
      "@type": "hydra:Collection",
      "hydra:member": [
        {
          "@id": "/books/1",
          "@type": "Book",
          "id": 1,
          "isbn": "9781782164104",
          "title": "Persistence in PHP with the Doctrine ORM",
          "description": "This book is designed for PHP developers and architects who want to modernize their skills through better understanding of Persistence and ORM.",
          "author": "K\u00e9vin Dunglas",
          "publicationDate": "2013-12-01T00:00:00+00:00",
          "reviews": []
        }
      ],
      "hydra:totalItems": 1
    }
    """

  Scenario: Throw errors when a post is invalid
    When I add "Content-Type" header equal to "application/ld+json"
    And I add "Accept" header equal to "application/ld+json"
    And I send a "POST" request to "/books" with body:
    """
    {
      "isbn": "1312",
      "title": "",
      "description": "Yo!",
      "author": "Me!",
      "publicationDate": "2016-01-01"
    }
    """
    Then the response status code should be 400
    And the response should be in JSON
    And the header "Content-Type" should be equal to "application/ld+json; charset=utf-8"
    And the JSON should be equal to:
    """
    {
      "@context": "/contexts/ConstraintViolationList",
      "@type": "ConstraintViolationList",
      "hydra:title": "An error occurred",
      "hydra:description": "isbn: This value is neither a valid ISBN-10 nor a valid ISBN-13.\ntitle: This value should not be blank.",
      "violations": [
        {
          "propertyPath": "isbn",
          "message": "This value is neither a valid ISBN-10 nor a valid ISBN-13."
        },
        {
          "propertyPath": "title",
          "message": "This value should not be blank."
        }
      ]
    }
    """

  # The "@dropSchema" annotation must be added on the last scenario of the feature file to drop the temporary SQLite database
  @dropSchema
    Scenario: Add a review
    When I add "Content-Type" header equal to "application/ld+json"
    When I add "Accept" header equal to "application/ld+json"
    And I send a "POST" request to "/reviews" with body:
    """
    {
      "rating": 5,
      "body": "Must have!",
      "author": "Foo Bar",
      "publicationDate": "2016-01-01",
      "book": "/books/1"
    }
    """
    Then the response status code should be 201
    And the response should be in JSON
    And the header "Content-Type" should be equal to "application/ld+json; charset=utf-8"
    And the JSON should be equal to:
    """
    {
      "@context": "/contexts/Review",
      "@id": "/reviews/1",
      "@type": "Review",
      "id": 1,
      "rating": 5,
      "body": "Must have!",
      "author": "Foo Bar",
      "publicationDate": "2016-01-01T00:00:00+00:00",
      "book": "/books/1"
    }
    """
```

Behat环境下的API Platform还附带了一个专门用于测试的临时SQLite数据库。它开箱即用。 

清除`测试`环境的缓存：

    $ docker-compose exec php bin/console cache:clear --env=test

然后运行:

    $ docker-compose exec php vendor/bin/behat

现在一切都应该是绿色透明的。现在，您可以通过Behat指定和测试您的Linked Data API！ 

您可能还对这些替代测试工具感兴趣（不包含在API Platform发行版中）：

* [Postman测试](https://www.getpostman.com/docs/writing_tests)（专有）：使用一个漂亮的UI为您的API Platform项目创建功能测试，从[Swagger集成](https://www.getpostman.com/docs/importing_swagger) 获益并且使用[newman](https://github.com/postmanlabs/newman)在CI中的尝试运行测试
* [PHP Matcher](https://github.com/coduo/php-matcher)：JSON文档测试的瑞士军刀

## 使用PHPUnit运行单元测试

要安装[PHPUnit](https://phpunit.de/) 测试套件，请执行以下命令:

    $ docker-compose exec php composer require --dev symfony/phpunit-bridge

要运行你的 [PHPUnit](https://phpunit.de/) 测试套件, 请执行以下命令:

    $ docker-compose exec php bin/phpunit
