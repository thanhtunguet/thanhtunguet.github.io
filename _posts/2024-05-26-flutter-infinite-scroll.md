---
title: "How to Create a Simple Infinite Scroll List in Flutter"
date: 2024-05-24 00:01:00 +0700
categories: [Mobile Development, Flutter]
tags: [Mobile Development, Flutter]
pin: false
---

Infinite scrolling is a common UI pattern that allows users to continuously load more content as they scroll, without the need for explicit pagination. In this article, we'll explore how to create a simple infinite scroll list in Flutter using the `infinite_scroll_pagination` package. We'll use a reusable template widget called `SimpleInfiniteList` that can be customized to fit various data models and repositories.

## Introduction

In this tutorial, we'll build a reusable Flutter widget, `SimpleInfiniteList`, which can handle infinite scrolling for any type of data model. This widget uses the `infinite_scroll_pagination` package to manage paging and data fetching efficiently.

## Prerequisites

Before we dive in, make sure you have the following set up:
1. Flutter SDK
2. A Flutter project
3. Dependencies: `infinite_scroll_pagination`, `truesight_flutter`

Add the dependencies in your `pubspec.yaml` file:

```yaml
dependencies:
  flutter:
    sdk: flutter
  infinite_scroll_pagination: ^3.1.0
  truesight_flutter: ^1.0.0
```

## Creating the `SimpleInfiniteList` Widget

Let's start by examining the code for the `SimpleInfiniteList` widget. This widget is a generic template that works with any data model, filter, and repository that you define.

### Widget Code

```dart
import 'package:flutter/material.dart';
import 'package:infinite_scroll_pagination/infinite_scroll_pagination.dart';
import 'package:truesight_flutter/truesight_flutter.dart';

class SimpleInfiniteList<T extends DataModel, TFilter extends DataFilter, TRepo extends BaseRepository<T, TFilter>>
    extends StatefulWidget {
  final Widget Function(BuildContext context, T item, int index) itemBuilder;
  final TFilter Function() filterInitializer;
  final TRepo simpleListRepo;

  const SimpleInfiniteList({
    super.key,
    required this.itemBuilder,
    required this.simpleListRepo,
    required this.filterInitializer,
  });

  @override
  State<StatefulWidget> createState() {
    return _SimpleInfiniteListState<T, TFilter, TRepo>();
  }
}

class _SimpleInfiniteListState<T extends DataModel, TFilter extends DataFilter,
    TRepo extends BaseRepository<T, TFilter>> extends State<SimpleInfiniteList<T, TFilter, TRepo>> {
  static const _pageSize = 20;
  final PagingController<int, T> _pagingController = PagingController<int, T>(firstPageKey: 0);
  late TFilter filter;

  @override
  void initState() {
    filter = widget.filterInitializer();
    _pagingController.addPageRequestListener((pageKey) {
      _handleFetchNewData(pageKey: pageKey);
    });
    super.initState();
  }

  _handleFetchNewData({int pageKey = 0}) async {
    filter.skip = pageKey;
    try {
      Future.wait([
        widget.simpleListRepo.list(filter),
        widget.simpleListRepo.count(filter),
      ]).then((values) {
        final List<T> list = values[0] as List<T>;
        final int count = values[1] as int;
        final isLastPage = list.length < _pageSize || list.length + (filter.skip!) == count;

        if (isLastPage) {
          _pagingController.appendLastPage(list);
        } else {
          final nextPageKey = pageKey + list.length;
          _pagingController.appendPage(list, nextPageKey);
        }
      });
    } catch (error) {
      _pagingController.error = error;
    }
  }

  Future<void> _onRefresh() async {
    await _handleFetchNewData(pageKey: 0);
  }

  @override
  Widget build(BuildContext context) {
    return RefreshIndicator(
      onRefresh: _onRefresh,
      child: PagedListView<int, T>(
        pagingController: _pagingController,
        builderDelegate: PagedChildBuilderDelegate<T>(
          itemBuilder: widget.itemBuilder,
        ),
      ),
    );
  }
}
```

### Breakdown of the Code

#### 1. Widget Definition

The `SimpleInfiniteList` widget is a stateful widget that takes three parameters:

- `itemBuilder`: A function that defines how to build each item in the list.
- `filterInitializer`: A function that initializes the filter object used for querying data.
- `simpleListRepo`: A repository instance that handles data fetching.

#### 2. State Class

The state class `_SimpleInfiniteListState` manages the paging controller and data fetching logic.

- **Paging Controller**: `_pagingController` is used to manage the paging state, including requesting new pages and handling errors.
- **Filter Initialization**: The `filter` object is initialized using `filterInitializer`.

#### 3. Data Fetching

The `_handleFetchNewData` method is responsible for fetching new data pages. It updates the filter with the current page key, requests data from the repository, and updates the paging controller with the new data.

- **Pagination Logic**: The method determines whether the fetched data is the last page and updates the paging controller accordingly.

#### 4. Refresh Indicator

The `build` method returns a `RefreshIndicator` that wraps a `PagedListView`. This setup allows users to pull down to refresh the list.

### Using the `SimpleInfiniteList`

To use the `SimpleInfiniteList` widget, you need to define your data model, filter, and repository. Here's an example:

```dart
class MyDataModel extends DataModel {
  // Define your data model properties
}

class MyDataFilter extends DataFilter {
  // Define your filter properties
}

class MyRepository extends BaseRepository<MyDataModel, MyDataFilter> {
  @override
  Future<List<MyDataModel>> list(MyDataFilter filter) {
    // Implement your data fetching logic
  }

  @override
  Future<int> count(MyDataFilter filter) {
    // Implement your count logic
  }
}

class MyInfiniteListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SimpleInfiniteList<MyDataModel, MyDataFilter, MyRepository>(
      itemBuilder: (context, item, index) {
        return ListTile(
          title: Text(item.name), // Customize as per your model
        );
      },
      filterInitializer: () => MyDataFilter(),
      simpleListRepo: MyRepository(),
    );
  }
}
```

## Conclusion

The `SimpleInfiniteList` widget provides a flexible and reusable solution for implementing infinite scrolling in Flutter applications. By leveraging the `infinite_scroll_pagination` package and a custom data repository, you can easily integrate infinite scroll functionality into your app. Customize the widget with your own data models and repositories to suit your specific needs.

Happy coding!
