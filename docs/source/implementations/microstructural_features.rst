.. _implementations-microstructural_features:

============================
Microstructural Features
============================

This module implements features from Chapter 18: Entropy features, Chapter 19: Microstructural features

.. image:: microstructural_features_images/kyle_lambda.png
   :scale: 50 %
   :align: center

Entropy features
================

Entropy is used to measure the average amount of information produced by a source of data. In financial machine learning, sources of data to get entropy from can be tick sizes, tick rule series,
percent changes between ticks. Estimating entropy requires the encoding of a message. Researcher can apply either binary (usually applied to tick rule), quantile or sigma encoding.

.. py:currentmodule:: mlfinlab.microstructural_features.encoding
.. automodule:: mlfinlab.microstructural_features.encoding
   :members: encode_tick_rule_array, quantile_mapping, sigma_mapping, encode_array

There are various ways to estimate entropy: Shannon, Lempel-Ziv, Plug-In, Kontoyiannis

.. py:currentmodule:: mlfinlab.microstructural_features.entropy
.. automodule:: mlfinlab.microstructural_features.entropy
   :members: get_shannon_entropy, get_lempel_ziv_entropy, get_plug_in_entropy

::

  from mlfinlab.entropy import get_shannon_entropy, get_lempel_ziv_entropy, get_plug_in_entropy
  message = 'abbnaacdeaas'
  shannon = get_shannon_entropy(message)
  lempel_ziv = get_lempel_ziv_entropy(message)
  plug_in = get_plug_in_entropy(message, word_length=1)

Bar based (inter-bar) features
==============================

When bars are generated (time, volume, imbalance, run) researcher can get inter-bar microstructural features: Roll Measure, Roll Impact, Corwin-Schultz spread estimator, Bekker-Parkinson volatility,
Kyle/Amihud/Hasbrouck lambdas, VPIN.

.. py:currentmodule:: mlfinlab.microstructural_features.first_generation
.. automodule:: mlfinlab.microstructural_features.first_generation
   :members: get_roll_measure, get_roll_impact, get_corwin_schultz_estimator, get_bekker_parkinson_vol

.. py:currentmodule:: mlfinlab.microstructural_features.second_generation
.. automodule:: mlfinlab.microstructural_features.second_generation
  :members: get_bar_based_kyle_lambda, get_bar_based_amihud_lambda, get_bar_based_hasbrouck_lambda

.. py:currentmodule:: mlfinlab.microstructural_features.third_generation
.. automodule:: mlfinlab.microstructural_features.third_generation
  :members: get_vpin

Trade based (intra-bar) features
================================

Some microstructural features need to be calculated from trades (tick rule/volume/percent change entropies, average tick size, vwap, tick rule sum, trade based lambdas). Mlfinlab has a special function
which calculates features for generated bars using trade data and bar date_time index.

.. py:currentmodule:: mlfinlab.microstructural_features.feature_generator
.. automodule:: mlfinlab.microstructural_features.feature_generator
  :members: MicrostructuralFeaturesGenerator

::

    import numpy as np
    import pandas as pd
    from mlfinlab.microstructural_features import quantile_mapping, MicrostructuralFeaturesGenerator

    df_trades = pd.read_csv('TRADES_PATH', parse_dates=[0])
    df_trades['log_ret'] = np.log(df_trades.Price / df_trades.Price.shift(1)).dropna()
    non_null_log_ret = df_trades[df_trades.log_ret != 0].log_ret.dropna()

    # Take unique volumes only
    volume_mapping = quantile_mapping(df_trades.Volume.drop_duplicates(), num_letters=10)

    returns_mapping = quantile_mapping(non_null_log_ret, num_letters=10)

    # Compress bars from ticks
    compressed_bars = pd.read_csv('BARS_PATH', index_col=0, parse_dates=[0])
    bar_index = compressed_bars.index

    gen = MicrostructuralFeaturesGenerator('TRADES_PATH', bar_index, volume_encoding=volume_mapping,
                                               pct_encoding=returns_mapping)
    features = gen.get_features(to_csv=False, verbose=False)
