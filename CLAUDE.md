# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このリポジトリは GitHub のマージキュー機能とスカッシュマージの動作を検証するためのものです。具体的には、[VOICEVOX/voicevox_engine#1734](https://github.com/VOICEVOX/voicevox_engine/issues/1734) で報告されている潜在的な問題を調査しています。

この問題は以下の状況で発生する可能性があります：

- 親設定（Pull Requests の設定）で「Allow merge commits」がオンの状態でマージキューのマージ方式を fast-forward merge に設定した後
- 親設定の「Allow merge commits」をオフにした場合、UI 上ではスカッシュマージと表示されるが、実際には fast-forward merge が実行される可能性がある

## リポジトリの目的

このリポジトリは以下を検証するためのテスト環境として機能します：

- マージ方法に関するリポジトリ設定の変更
- 異なる設定下でのマージキューの動作確認
- 期待される動作と実際のマージ結果の比較検証
