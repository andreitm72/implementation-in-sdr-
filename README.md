// Changes to implement the auditory green line in the worker function of the scanner plugin

#include <chrono>
#include <iostream>
#include "plugin/scanner.h"
#include "flog.h"

namespace plugin {

void Scanner::worker() {
    using namespace std::chrono;

    auto now = steady_clock::now();

   // Threshold defined by the slider (adjustable green line)
    const float level = config.scanner.level;

    flog::info("Scanner level set: {} dB", level);

    while (running) {
        auto spectrum = getSpectrumData();

        if (!spectrum.empty()) {
            float maxLevel = getMaxLevel(spectrum);

            if (maxLevel >= level) {
                // Signal above the threshold of the green line -> audible
                flog::info("Semnal detectat peste prag: {} dB", maxLevel);
                lastSignalTime = now;

                // Existing logic for audio signal processing
                processAudibleSignal(spectrum);
            } else {
                // Signal below threshold -> we ignore auditory
                flog::info("Semnal sub prag: {} dB, tăiat auditiv", maxLevel);
            }
        }

        std::this_thread::sleep_for(milliseconds(50)); // Scan rate control
    }
}

void Scanner::setupGui() {
    // Interface for the green line slide
    if (ImGui::SliderFloat("##scanner_level", &config.scanner.level, -150.0f, 0.0f, "Prag: %.1f dB")) {
        flog::info("Prag ajustat la: {} dB", config.scanner.level);
    }
}

} // namespace plugin
