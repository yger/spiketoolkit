
Sorter Tutorial
===============

This notebook shows how to use the spiketoolkit.sorters module to:


1. check available sorters
2. check and set sorters parameters
3. run sorters
4. use the spike sorter launcher
5. spike sort by property


.. code:: python

    import spikeextractors as se
    import spiketoolkit as st
    import spikewidgets as sw
    import os
    import time
    from pprint import pprint

First, let's create a toy example:

.. code:: python

    recording, sorting_true = se.example_datasets.toy_example(duration=60)

1) Check available sorters
--------------------------

.. code:: python

    print(st.sorters.available_sorters())


.. parsed-literal::

    ['herdingspikes', 'ironclust', 'kilosort', 'kilosort2', 'klusta', 'mountainsort4', 'spykingcircus', 'tridesclous']


This will list the sorters installed in the machine. Each spike sorter
is implemented in a class. To access the class names you can run:

.. code:: python

    st.sorters.installed_sorter_list




.. parsed-literal::

    [spiketoolkit.sorters.klusta.klusta.KlustaSorter,
     spiketoolkit.sorters.tridesclous.tridesclous.TridesclousSorter,
     spiketoolkit.sorters.mountainsort4.mountainsort4.Mountainsort4Sorter,
     spiketoolkit.sorters.ironclust.ironclust.IronclustSorter,
     spiketoolkit.sorters.kilosort.kilosort.KilosortSorter,
     spiketoolkit.sorters.kilosort2.kilosort2.Kilosort2Sorter,
     spiketoolkit.sorters.spyking_circus.spyking_circus.SpykingcircusSorter,
     spiketoolkit.sorters.herdingspikes.herdingspikes.HerdingspikesSorter]



2) Check and set sorters parameters
-----------------------------------

To check which parameters are available for each spike sorter you can
run:

.. code:: python

    default_ms4_params = st.sorters.Mountainsort4Sorter.default_params()
    pprint(default_ms4_params)


.. parsed-literal::

    {'adjacency_radius': -1,
     'clip_size': 50,
     'curation': True,
     'detect_interval': 10,
     'detect_sign': -1,
     'detect_threshold': 3,
     'filter': False,
     'freq_max': 6000,
     'freq_min': 300,
     'noise_overlap_threshold': 0.15,
     'whiten': True}


Parameters can be changed either by passing a full dictionary, or by
passing single arguments.

.. code:: python

    # Mountainsort4 spike sorting
    default_ms4_params['detect_threshold'] = 4
    default_ms4_params['curation'] = False
    
    # parameters set by params dictionary
    sorting_MS4 = st.sorters.run_mountainsort4(recording=recording, **default_ms4_params, 
                                               output_folder='tmp_MS4')


.. parsed-literal::

    ...

.. code:: python

    # parameters set by params dictionary
    sorting_MS4_10 = st.sorters.run_mountainsort4(recording=recording, detect_threshold=10, 
                                               output_folder='tmp_MS4')


.. parsed-literal::

    ...


.. code:: python

    print('Units found with threshold = 4:', sorting_MS4.get_unit_ids())
    print('Units found with threshold = 10:', sorting_MS4_10.get_unit_ids())


.. parsed-literal::

    Units found with threshold = 4: [ 1  2  3  4  5  6  7  8  9 10 11 12 13 14 15]
    Units found with threshold = 10: [1 2 3]


3) Run sorters
--------------

.. code:: python

    # SpyKING Circus spike sorting
    sorting_SC = st.sorters.run_spykingcircus(recording, output_folder='tmp_SC')
    print('Units found with Spyking Circus:', sorting_SC.get_unit_ids())

.. code:: python

    # KiloSort spike sorting (KILOSORT_PATH and NPY_MATLAB_PATH can be set as environment variables)
    sorting_KS = st.sorters.run_kilosort(recording, output_folder='tmp_KS')
    print('Units found with Kilosort:', sorting_KS.get_unit_ids())

.. code:: python

    # Kilosort2 spike sorting (KILOSORT2_PATH and NPY_MATLAB_PATH can be set as environment variables)
    sorting_KS2 = st.sorters.run_kilosort2(recording, output_folder='tmp_KS2')
    print('Units found with Kilosort2', sorting_KS2.get_unit_ids())

.. code:: python

    # Klusta spike sorting
    sorting_KL = st.sorters.run_klusta(recording, output_folder='tmp_KL')
    print('Units found with Klusta:', sorting_KL.get_unit_ids())

.. code:: python

    # IronClust spike sorting (IRONCLUST_PATH can be set as environment variables)
    sorting_IC = st.sorters.run_ironclust(recording, output_folder='tmp_IC')
    print('Units found with Ironclust:', sorting_IC.get_unit_ids())

.. code:: python

    # Tridesclous spike sorting
    sorting_TDC = st.sorters.run_tridesclous(recording, output_folder='tmp_TDC')
    print('Units found with Tridesclous:', sorting_TDC.get_unit_ids())

4) Use the spike sorter launcher
--------------------------------

The launcher enables to call any spike sorter with the same functions:
``run_sorter`` and ``run_sorters``. For running multiple sorters on the
same recording extractor or a collection of them, the ``run_sorters``
function can be used.

.. code:: python

    st.sorters.run_sorters?

.. code:: python

    recording_list = [recording]
    sorter_list = ['klusta', 'mountainsort4', 'tridesclous']

.. code:: python

    sorting_output = st.sorters.run_sorters(sorter_list, recording_list, working_folder='working')


.. parsed-literal::

    ...


.. code:: python

    for sorter, extractor in sorting_output['recording_0'].items():
        print(sorter, extractor.get_unit_ids())


.. parsed-literal::

    klusta [0, 2, 3, 4, 5, 6, 7]
    mountainsort4 [ 2  3  5  6  7 10 11 16]
    tridesclous [0, 1, 2, 3, 4]


5) Spike sort by property
-------------------------

Sometimes, you might want to sort your data depending on a specific
property of your recording channels.

For example, when using multiple tetrodes, a good idea is to sort each
tetrode separately. In this case, channels belonging to the same tetrode
will be in the same 'group'. Alternatively, for long silicon probes,
such as Neuropixels, you could sort different areas separately, for
example hippocampus and thalamus.

All this can be done by sorting by 'property'. Properties can be loaded
to the recording channels either manually (using the
``set_channel_property`` method, or by using a probe file. In this
example we will create a 16 channel recording and split it in four
tetrodes.

.. code:: python

    recording_tetrodes, sorting_true = se.example_datasets.toy_example(duration=60, num_channels=16)
    
    # initially there is no group information
    print(recording_tetrodes.get_channel_property_names())


.. parsed-literal::

    ['location']


.. code:: python

    # working in linux only
    !cat tetrode_16.prb


.. parsed-literal::

    channel_groups = {
        0: {
            'channels': [0,1,2,3],
        },
        1: {
            'channels': [4,5,6,7],
        },
        2: {
            'channels': [8,9,10,11],
        },
        3: {
            'channels': [12,13,14,15],
        }
    }


.. code:: python

    # load probe file to add group information
    recording_tetrodes = se.load_probe_file(recording_tetrodes, 'tetrode_16.prb')
    print(recording_tetrodes.get_channel_property_names())


.. parsed-literal::

    ['group', 'location']


We can now use the launcher to spike sort by the property 'group'. The
different groups can also be sorted in parallel, and the output sorting
extractor will have the same property used for sorting. Running in
parallel can speed up the computations.

.. code:: python

    t_start = time.time()
    sorting_tetrodes = st.sorters.run_sorter('klusta', recording_tetrodes, output_folder='tmp_tetrodes', 
                                             grouping_property='group', parallel=False)
    print('Elapsed time: ', time.time() - t_start)


.. parsed-literal::

    Elapsed time:  11.47568941116333


.. code:: python

    t_start = time.time()
    sorting_tetrodes_p = st.sorters.run_sorter('klusta', recording_tetrodes, output_folder='tmp_tetrodes', 
                                               grouping_property='group', parallel=True)
    print('Elapsed time parallel: ', time.time() - t_start)

.. code:: python

    print('Units non parallel: ', sorting_tetrodes.get_unit_ids())
    print('Units parallel: ', sorting_tetrodes_p.get_unit_ids())

Now that spike sorting is done, it's time to do some postprocessing,
comparison, and validation of the results!
