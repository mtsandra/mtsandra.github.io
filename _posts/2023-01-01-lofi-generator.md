---
layout: post
title:  How I Built a Lofi Music Player with AI-Generated Tracks
date:   2023-01-02
tags: tech music
categories: music-tech-project tech-tutorial
description: 'AI solo tracks are generated with LSTM model and song tracks are made with Tone.js.'
---

<div style = 'font-family: "Lucida Console", "Courier New", monospace; font-size:  100%;'>

Try out the web player <a href='https://mtsandra.github.io/lofi-station'>here</a>! <br>
<!-- demo gif -->
<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/blog2023/lofi/lofi-station.gif" alt="lofi demo" width = "100%" style="padding-bottom:0.5em;" /><br>
Lofi Station Demo
</div>


<h3> Introduction </h3>

Lofi hip hop music has been my go-to study buddy ever since college. It creates a cozy and calming vibe with a relatively simple musical structure. Some jazzy chord progressions, groovy drum patterns, ambience sounds, and nostalgic movie quotes can give us a pretty decent sounding lofi hip hop track. On top of the musical side, animated visuals are also a crucial part of the lofi aesthetics, setting the ambience along with the nature sounds of water, wind, and fire. <br><br>

The idea to create my own lofi web player occurred to me on one Sunday afternoon when I was learning about deep generative models. I did some research and finished the project during the holiday times. The web player provides two options: users can either choose a lofi track based on a real song encoded with Tone.js, or they could choose an AI-generated solo track. Both options will be layered on top of the drum loop, ambience sounds, and quotes that users selected in the previous step. This post will mainly talk about how to use LSTM models to generate a midi track, and I will briefly discuss how to make a song with Tone.js at the end. <br><br>

<h3>LSTM Model Architecture & Midi Generation</h3>
In a previous <a href='/blog/2022/dl4mir-4'>post</a>, I explained what an LSTM network is. For a quick refresher, it is a special type of RNN that handles long term dependencies better. It also has a recurrent structure that takes the output from the previous timestep at the current timestep. To better understand it, we can unroll the network and think of an LSTM cell as multiple copies of the same network, each passing a message to the next timestep, as shown below.

<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/dl4mir/lstm-unrolled.png" alt="lofi demo" width = "100%" style="padding-bottom:0.5em;" /><br>
Unrolled LSTM Model Architecture
</div>
<br>

Each cell contains four main components that allow them to handle long term dependencies better:

<li> forget gate: determines what information to forget </li>
<li> input gate: determines what information to update and store in our cell state</li>
<li> cell state update: make element-wise operations to update the cell state</li>
<li> output gate: determines what information to output </li>

<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/dl4mir/lstm-components.jpeg" alt="lofi demo" width = "100%" style="padding-bottom:0.5em;" /><br>
Inside an LSTM cell
</div>

<h4>Training Data Preparation</h4>

We have a couple of options when it comes to the music data format we are training the model on: raw audio, audio features (e.g. time frequency representations like Mel spectrogram), or symbolic music representation (e.g. midi files). Our goal is to generate a solo track (i.e. a sequence of notes, chords, and rests) to layer on other components like drum loops, so <a style='font-weight: 600;'>midi files</a> are the easiest and most effective format to achieve our goal. Raw audio is very computationally expensive to train on. To put it in perspective, music clips sampled at 48000kHz mean there are 48000 data points in one second of audio. Even if we downsample it to 8kHz, that is still 8000 data points for every second. In addition, clean audio of only the melody or chord progression is extremely rare. However, we could still find some midi files containing only the chord progression / melody if we try hard enough. <br><br>

For this project, I found some free lofi midi samples on YouTube and sample download websites. I also leveraged some midi files in this Kaggle <a href='https://www.kaggle.com/code/zakarii/lo-fi-hip-hop-generation'>dataset</a>. In order to make sure that I am training my model on quality data (plausible chord progression and meter for lofi hip hop), I listened to a subset of tracks from each source and filtered out my training dataset. The model architecture is inspired by the classical piano composer repository <a href='https://github.com/Skuldur/Classical-Piano-Composer'>here</a>.<br><br>

<h5>Load and encode the midi files</h5>

A Python package <a href='http://web.mit.edu/music21/doc/index.html'>music21</a> can be used to load the midi files. Music21 parses a midi file and stores each musical component into a specific Python object. In other words, a note will be saved as a Note object, a chord will be saved as a Chord object, and a rest will be saved as a Rest object. Their name, duration, pitch class and other attributes can be accessed through the dot notation. Music21 stores the music clip in a hierarchy shown below, and we can extract the necessary information accordingly. If you are interested in how to use the package, the package website has a beginner-friendly user guide and Valerio Velardo from The Sound of AI has a <a href='https://www.youtube.com/watch?v=coEgwnMBuo0&ab_channel=ValerioVelardo-TheSoundofAI'>tutorial</a> on how to use music21  as well. 

<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/blog2023/lofi/music21_hierarchy.jpeg" alt="music21 hierarchy" width = "100%" style="padding-bottom:0.5em;" /><br>
Music21 hierarchy
</div>

As mentioned previously, music21 stores each note, rest, and chord as a Python object, so the next step is to encode them and map them to integers that the model can train on. The model output should contain not just notes, but also chords and rests, so we will encode each type separately and map the encoded value to an integer. We do this for all the midi files and concatenate them into one sequence to train the model on.

<li>Chords: get the pitch names of the notes in a chord, convert them to their normal order and connect them with a dot in the string format, "#.#.#"</li>
<li>Notes: use the pitch name as the encoding</li>
<li>Rests: encoded as the string "r"</li>
<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/blog2023/lofi/encoding.PNG" alt="encoding" width = "100%" style="padding-bottom:0.5em;" /><br>
Midi Encoding and Mapping
</div>

<h5>Create the Input and Target Pairs</h5>

Now we have a model-friendly encoding of our midi data. The next step would be to prepare the input and target pairs to feed into the model. In a simple supervised classification machine learning problem, there are input and target pairs. For example, a model that classifies a dog breed will have the fur color, height, weight, and eye color of the dog as input and the label / target will be the specific dog breed the dog belongs to. In our case, the input will be a sequence of length k starting from timestep i, and the corresponding target will be the data point at timestep i+k. So we will loop through our encoded note sequence and create the input and target pairs for the model. As a last step, we change the dimension of the input to the keras-friendly format and one-hot encode the output. <br><br>

<h4>Model Structure</h4>
As previously mentioned, we will use LSTM layers as our core structure. In addition, this network also uses the below components:

<li>Dropout layers: to regularize the network and prevent overfitting by randomly setting input units to 0 with a frequency of rate at each step during training time (in our case, the rate is 0.3)</li>
<li>Dense layers: fully connects the preceding layer and performs matrix-vector multiplication in each node. The last dense layer needs to have the same amount of nodes as the total number of unique notes/chords/rest in our network. </li>
<li>Activation layers: adds non-linearity to our network if used in a hidden layer and helps the network classify to a class if used in an output layer. </li>
<code><pre>
model = Sequential()
model.add(LSTM(
    256,
    input_shape=(network_input.shape[1], network_input.shape[2]),
    return_sequences=True
))
model.add(Dropout(0.3))
model.add(LSTM(512, return_sequences=True))
model.add(Dropout(0.3))
model.add(LSTM(256))
model.add(Dense(256))
model.add(Dropout(0.3))
model.add(Dense(n_vocab))
model.add(Activation('softmax'))
model.compile(loss='categorical_crossentropy', optimizer='rmsprop')
model.fit(network_input, network_output, epochs=200, batch_size=128)
</pre>
</code>
For this example network, 3 LSTM layers are used with 2 dropout layers each following the first two LSTM layers. Then there are 2 fully connected dense layers, followed by one softmax activation function. Since the outputs are categorical, we will use categorical cross entropy as the loss function. The RMSprop optimizer is used, which is pretty common for RNNs. Checkpoints are also added so that weights are regularly saved at different epochs and could be used before the model finishes training. Please feel free to tweak the model structure, and try with different optimizers, epochs and batch sizes. <br><br>

<h4>Output Generation & Decoding Back to Midi Notes</h4>

The output generation process is similar to the training process – we give the model a sequence of length m (we'll also call it sequence m for notation simplification) and ask it to predict the next data point. This sequence m has a start index randomly selected from the input sequence, but we can also specify a specific start index if we wish. The model output is a list of probabilities from softmax that tell us how much each class is suited as the next data point. We will pick the class with the highest probability.  In order to generate a sequence of length j, we will repeat this process by removing the first element of the sequence m and adding the recently generated data point to this sequence m, until the model generates j new data points. <br><br>

The data generated from the last paragraph is still an integer, so we decode it back to a note/chord/rest using the same mappings during encoding. If it is a chord string format, we will read the integer notation from the string "#.#.#.#" and create a music21.chord object. If it is a note or rest, we will create a corresponding note and rest object accordingly. At the same time, we append the new data point generated to the prediction output sequence at each timestep. For an illustration of this process, please see the example flow below where we are generating a sequence of 4 data points with an input sequence of 3 data points.


<div style="font-size: 80%; color: #808080; text-align:center; font-style: italic; font-family: 'Lucida Console', 'Courier New', monospace;">
<img src="/assets/img/blog2023/lofi/midi-generation.PNG" alt="decoding" width = "100%" style="padding-bottom:0.5em;" /><br>
Output Generation and Midi Decoding
</div>

Now we have a sequence of notes, chords, and rests. We could put them in a music21 stream and write out the midi file, in which case all the notes will be quarter notes. To keep the output a little bit more interesting, I added a code snippet that randomly samples a duration to specify for each note or chord (The default probability distribution is 0.65 for eighth notes, 0.25 for 16th notes, 0.05 for both quarter and half notes). Rests are defaulted to 16th rests so that we don't have too long of a silence between notes.  
<code><pre>
NOTE_TYPE = {
            "eighth": 0.5,
            "quarter": 1,
            "half": 2,
            "16th": 0.25
        }
offset = 0
output_notes = []

for pattern in prediction_output:
    curr_type = numpy.random.choice(list(NOTE_TYPE.keys()), p=[0.65,0.05,0.05, 0.25])
    
    # pattern is a chord
    if ('.' in pattern) or pattern.isdigit():
        notes_in_chord = pattern.split('.')
        notes = []
        for current_note in notes_in_chord:
            new_note = note.Note(int(current_note))
            new_note.storedInstrument = instrument.Piano()
            notes.append(new_note)
        new_chord = chord.Chord(notes, type=curr_type)
        new_chord.offset = offset
        output_notes.append(new_chord)
    elif str(pattern).upper() == "R":
        curr_type = '16th'
        new_rest = note.Rest(type=curr_type)
        new_rest.offset = offset
        output_notes.append(new_rest)
    # pattern is a note
    else:
        new_note = note.Note(pattern, type=curr_type)
        new_note.offset = offset
        new_note.storedInstrument = instrument.Piano()
        output_notes.append(new_note)

    # increase offset each iteration so that notes do not stack
    offset += NOTE_TYPE[curr_type]

midi_stream = stream.Stream(output_notes)

midi_stream.write('midi', fp='test_output.mid')
</pre></code>

Once we run the model for a few times with different parameters and pick out the tracks that we like, we will choose a lofi-esque instrument effect in any DAW so that our generated tracks sound more like real music. Then we head over to JavaScript to build our web player. 
<br><br>

<h3>Building the web player with <a href='https://tonejs.github.io/'>Tone.js</a></h3>

Tone.js is a web audio framework for creating interactive music in the browser. You can use it to build a lot of fun interactive websites (see <a href='https://tonejs.github.io/demos'>demos</a> here). But in our case, it provides a global transport to make sure our drum patterns, ambience sounds, quotes, and melody play at the same time. It also allows us to write in music score, sample an instrument, add sound effects (reverberation, gain, etc.), and create loops right in JavaScript. Credit goes to <a href='https://github.com/lawreka/loaf-ai'>Kathryn</a> for the code skeleton. If you want a quick and effective crash course on Tone.js, I highly recommend the use case examples on their <a href='https://tonejs.github.io'>website</a>. The most important takeaway is that for each sound event we create, we need to connect them to the AudioDestinationNode (aka our speakers) through <code>toDestination()</code> or through <code>samplePlayer.chain(effect1, Tone.Destination)</code> if we want to add sound effects to it. Then through <code>Tone.Transport</code>, we will be able to start, pause, and schedule events on the master output. <br><br>

<h5>Looping the Audio Clips</h5>
Drum patterns, ambience sounds, quotes, and the pre-generated AI tracks are all audio files (.mp3 or .wav) loaded into our web player through a Player class. After loading the user input events from the website, they are then fed into a Tone.js Part class to create loops. 
<br><br>
Drum patterns are looped every 8 measures, ambience sounds every 12 measures, and AI solo tracks every 30 measures. Quotes are not looped and start at the beginning of the 5th measure. <br><br>

<h5>Creating Melody and Chord Progression with Instrument Samples</h5>
Tone.js does not provide software instrument options that we see in DAW, only samplers that allow us to sample our own instruments by loading in a couple of notes. The sampler will then repitch the samples automatically to create the pitches which were not explicitly included. 
<br><br>
Then we can write in the melody and chord progression by specifying the notes and the time that the note should take place. I recommend using TransportTime to encode the beat exactly as we want. TransportTime is in the form of "BARS:QUARTERS:SIXTEENTHS" and uses zero-based numbering. For example, "0:0:2" means the note will take place after two sixteenth notes passed in the first bar. "2:1:0" means the note will take place in the third bar, after one quarter note passed. I wrote in the melody and chord progressions for 3 existing songs: Ylang Ylang by FKJ, La Festin by Camille, and See You Again by Tyler, the Creator this way. 

<br><br>

<h5>web player Design</h5>
I added functions to change the web player background with different inputs for ambience sounds so that a more context-appropriate gif will be displayed for each context. There is also a visualizer connected with the song notes to add visual appeal, made with p5.js. <br><br>

<h3>Future Work</h3>

<h5>The LSTM Model</h5>
<li>Add start of sequence and end of sequence tokens so that the model can learn the music pattern as the song comes to an end.</li>
<li>Incorporate encoding for note duration so that beat tracking can be enabled.</li>
<br>
<h5>The web player </h5>
<li>It would be cool to connect the backend AI model to the web player so that the output can be generated live. Currently the roadblock is that the model takes a few minutes to generate the output, but likely we could leverage a pre-trained model API. </li>
<li>User interactivity could be greatly improved if the web player allows the users to 1) put in chord progression of their own choice 2) write down some text that the web player will do sentiment analysis on and output a chord progression matching the sentiment.    </li>
<br><br>

<h3>Conclusion</h3>

For code and training datasets, please see the GitHub repo <a href='https://github.com/mtsandra/lofi-station'>here</a>. While this is just a simple lofi web player, I have had a lot of fun with both the LSTM model and Tone.js. It amazes me every time to see how we can incorporate technology into our experience with music. 

</div>
