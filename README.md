# parallel-pytest
[![CircleCI](https://dl.circleci.com/status-badge/img/gh/mfunaki-circleci/parallel-pytest/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/mfunaki-circleci/parallel-pytest/tree/main)
## テスト分割によるパイプラインの高速化
このリポジトリは、テストの自動分割によるパイプラインの高速化の一例です。

![テストを並行実行して完了時間を短縮](/images/fig-parallel-circleci.png)

ディレクトリには pytest を使ったテストファイルが10個あり、テスト完了までにそれぞれ、15秒、50秒、5秒...20秒かかります(実際にはsleepしているだけ)。

ここで、pytest.ini ファイルには、次のような記述をしておきます。
```pytest.ini
[pytest]
junit_family=legacy
```
これにより、テストの実行結果をJUnit互換形式で出力しており、その実行結果を次のようにconfig.ymlに記述しておくことで、CircleCI内に保持しています。
```config.yml
      - store_test_results:
          path: test-results
```

１並列で実行した場合、6分20秒程度かかります(CircleCIは個々のテストファイルの実行時間を認識しています)。
```config.yml
  build-and-test:
    parallelism: 1
```

それでは、３並列にしてみます。
```config.yml
  build-and-test:
    parallelism: 3
```
前回のテスト結果をもとに、３コンテナで実行時間が均等になるように、つまり実行時間が３分の１になるように、テストが並列実行されます。

一方で、５並列で実行しても、実行時間は５分の１にはなりません。これは、hundred_second_test.pyの実行に100秒かかっているからです。実行時間を５分の１にするには、このテストファイルを(手動で)分割し、分割したテストファイルが76秒以下で終了するようにする必要があります。

このように、テストの自動分割によるパイプラインの高速化は、費用はほとんど変わらず、テスト完了時間を劇的に短縮することができますが、個々のテストファイルの最長実行時間以下には短縮されないことに注意が必要です。

- [テスト分割によるパイプラインの高速化](https://circleci.com/docs/ja/test-splitting-tutorial/)

## 失敗したテストのみを再実行する
10個あるテストファイルのうち、forty_second_flaky_test.py に関しては、50%の確率で失敗します。
テストに失敗した際に、10個のテストをすべて再実行しなくても、失敗したテストだけを再実行することができます。
![Rerun failed test](/images/rerun-failed-test-circleci.png)

これにより、再試行するテストの実行時間(今回の例でいえば40秒)でテストを完了することができます。

- [失敗したテストのみを再実行する](https://circleci.com/docs/ja/rerun-failed-tests/)

## テストインサイトによる結果が不安定なテスト(flakyなテスト)の検出

- [テストインサイト](https://circleci.com/docs/ja/insights-tests/)

## プロジェクトが suspend された場合
このプロジェクトでは、テストは(実際にテストを実行しているのではなく)sleepを実行しています。
その結果、CPUリソースを無駄に消費していると認識され、次のようなエラー(We did not run this pipeline because the project has been susbended)が出力され、ワークフローが実行できなくなることがあります。
![the project has been suspended](/images/suspended-circleci.png)
この場合、エラーメッセージにも書かれていますが、サポートにお問い合わせいただくことで、suspend 状態を解除することができます(Freeプランであってもサポートの利用は可能です)。日本語でお問い合わせいただけます。
- [サポートセンター](https://support.circleci.com/hc/ja)