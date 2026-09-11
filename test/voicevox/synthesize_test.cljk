(ns voicevox.synthesize-test
  (:require [kotoba.lang.text :as str]
            [clojure.test :refer [deftest is testing]]
            [voicevox.synthesize :as synth]))

(defn- stub [query-body]
  (fn [{:keys [url body]}]
    (cond
      (str/includes? url "/audio_query")
      {:status 200 :body-bytes (.getBytes query-body "UTF-8")}
      (str/includes? url "/synthesis")
      {:status 200 :body-bytes (.getBytes (str "WAV:" body) "UTF-8")})))

(deftest synthesize-test
  (testing "applies speed/pitch overrides onto the audio_query result before synthesis"
    (let [calls (atom [])
          http-fn (fn [req] (swap! calls conj req) ((stub "{\"speedScale\":1.0,\"pitchScale\":0.0}") req))
          wav (synth/synthesize! {:text "こんにちは" :style-id 2 :speed 0.9 :pitch 0.1}
                                 {:http-fn http-fn :base-url "http://x"})
          synthesis-body (String. ^bytes wav "UTF-8")]
      (is (str/includes? synthesis-body "\"speedScale\":0.9"))
      (is (str/includes? synthesis-body "\"pitchScale\":0.1"))))

  (testing "does not override when speed/pitch are default"
    (let [http-fn (stub "{\"speedScale\":1.0,\"pitchScale\":0.0}")
          wav (synth/synthesize! {:text "hi" :style-id 2} {:http-fn http-fn :base-url "http://x"})
          body (String. ^bytes wav "UTF-8")]
      (is (not (str/includes? body "0.9")))))

  (testing "blank text raises without calling the engine"
    (is (thrown? clojure.lang.ExceptionInfo
                 (synth/synthesize! {:text "" :style-id 2} {:http-fn (stub "{}")})))))
