---
title: "React Native + Expo で録音する"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["reactnative","expo"]
published: false
---

記事が少なかったので、サンプルを作ってみました。

```tsx
import { Audio } from 'expo-av';
import React from 'react';
import { View, Button } from 'react-native';

import type { FC } from 'react';

export const AudioSample: FC = () => {
  const [recording, setRecording] = React.useState<Audio.Recording | undefined>(undefined);
  const startRecording = async (): Promise<void> => {
    try {
      console.log('Requesting permissions to use mike');
      // マイクを利用するためのパーミッションを要求
      await Audio.requestPermissionsAsync();
      await Audio.setAudioModeAsync({
        allowsRecordingIOS: true,
        playsInSilentModeIOS: true,
      });

      console.log('Starting recording');
      // Recordingオブジェクトを作成し、録音を開始
      const recording = new Audio.Recording();
      await recording.prepareToRecordAsync(Audio.RECORDING_OPTIONS_PRESET_HIGH_QUALITY);
      await recording.startAsync();

      // recording 自体を状態として保持
      setRecording(recording);
      console.log('Recording started');
    } catch (error) {
      console.error('Failed to start recording', error);
    }
  };

  const stopRecording = async (): Promise<void> => {
    console.log('Stopping recording');
    if (!recording) {
      console.log('No recording to stop!');
      return;
    }
    await recording.stopAndUnloadAsync();
    // 録音が終了した状態のrecordingオブジェクトから、録音ファイルのURIを取得できる
    const uri = recording.getURI();
    setRecording(undefined);
    console.log('Recording stopped and stored at', uri);

  };

  return (
    <View>
      <Button
        // recording の状態によってボタンと関数を出し分ける
        title={recording ? 'Stop Recording' : 'Start Recording'}
        onPress={recording ? stopRecording : startRecording}
      />
    </View>
  );
};
```