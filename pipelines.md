# DNW Output Pipelines

| Category                   | Description                                | Keywords                                            | Toolchain Factory                             | Version |
|----------------------------|--------------------------------------------|-----------------------------------------------------|-----------------------------------------------|:-------:|
| **Debugging & Diagnostic** | Plain Text Full Manuscript                 | `plain`, `text`, `full`                             | `PlainTextManuscriptToolchain`                |  v2.0   |
|                            | Diagnostic Manuscript                      | `diagnostic`                                        | `DiagnosticManuscriptToolchain`               |  v2.0   |
| **Online Retailer**        | EPUB Full Manuscript for Direct            | `epub`, `direct`, `full`                            | `EpubDirectManuscriptToolchain`               |  v2.0   |
|                            | EPUB Sampler Manuscript for Direct         | `epub`, `direct`, `sampler`                         | `EpubDirectSamplerToolchain`                  |  v2.0   |
|                            | EPUB Full Manuscript for Amazon            | `epub`, `amazon`, `full`                            | `EpubAmznManuscriptToolchain`                 |  v2.0   |
|                            | EPUB Sampler Manuscript for Amazon         | `epub`, `amazon`, `sampler`                         | `EpubAmznSamplerToolchain`                    |  v2.0   |
|                            | EPUB Full Manuscript for Apple             | `epub`, `apple`, `full`                             | `EpubAaplManuscriptToolchain`                 |  v2.0   |
|                            | EPUB Sampler Manuscript for Apple          | `epub`, `apple`, `sampler`                          | `EpubAaplSamplerToolchain`                    |  v2.0   |
|                            | EPUB Full Manuscript for Google            | `epub`, `google`, `full`                            | `EpubGoogManuscriptToolchain`                 |  v2.0   |
|                            | EPUB Sampler Manuscript for Google         | `epub`, `google`, `sampler`                         | `EpubGoogSamplerToolchain`                    |  v2.0   |
|                            | EPUB Full Manuscript for Kobo              | `epub`, `kobo`, `full`                              | `EpubKoboManuscriptToolchain`                 |  v2.0   |
|                            | EPUB Sampler Manuscript for Kobo           | `epub`, `kobo`, `sampler`                           | `EpubKoboSamplerToolchain`                    |  v2.0   |
|                            | EPUB Full Manuscript for B&N Press         | `epub`, `bnp`, `full`                               | `EpubNookManuscriptToolchain`                 |  v2.0   |
|                            | EPUB Sampler Manuscript for B&N Press      | `epub`, `bnp`, `sampler`                            | `EpubNookSamplerToolchain`                    |  v2.0   |
|                            | EPUB Full Manuscript for ARC               | `epub`, `arc`, `full`                               | `EpubArcManuscriptToolchain`                  |  v2.0   |
|                            | EPUB Sampler Manuscript for ARC            | `epub`, `arc`, `sampler`                            | `EpubArcSamplerToolchain`                     |  v2.0   |
|                            | PDF Ebook Full Manuscript for ARC          | `ebook`, `pdf`, `arc`, `full`                       | `EbookPdfArcManuscriptToolchain`              |  v2.0   |
|                            | PDF Ebook Sampler Manuscript for ARC       | `ebook`, `pdf`, `arc`, `sampler`                    | `EbookPdfArcSamplerToolchain`                 |  v2.0   |
| **Social Media (Image)**   | Serialization for LinkedIn                 | `pdf`, `serial`, `linkedin`, `social`               | `LinkedInPdfSerializationToolchain`           |  v2.0   |
|                            | Serialization for Instagram                | `png`, `serial`, `instagram`, `social`              | `InstagramPngSerializationToolchain`          |  v2.0   |
|                            | Serialization for Threads                  | `png`, `serial`, `threads`, `social`                | `ThreadsPngSerializationToolchain`            |  v2.0   |
|                            | Serialization for Bluesky                  | `png`, `serial`, `bluesky`, `social`                | `BlueskyPngSerializationToolchain`            |  v2.0   |
|                            | Serialization for Twitter                  | `png`, `serial`, `twitter`, `social`                | `TwitterPngSerializationToolchain`            |  v2.0   |
|                            | Serialization for Facebook                 | `png`, `serial`, `facebook`, `social`               | `FacebookPngSerializationToolchain`           |  v2.0   |
|                            | Serialization for Tumblr                   | `png`, `serial`, `tumblr`, `social`                 | `TumblrPngSerializationToolchain`             |  v2.0   |
|                            | Serialization for Imgur                    | `png`, `serial`, `imgur`, `social`                  | `ImgurPngSerializationToolchain`              |  v2.0   |
|                            | Serialization for Pinterest                | `png`, `serial`, `pinterest`, `social`              | `PinterestPngSerializationToolchain`          |  v2.0   |
|                            | Serialization for Substack Notes           | `png`, `serial`, `substack`, `social`               | `SubstackPngSerializationToolchain`           |  v2.0   |
|                            | Serialization for Substack Newsletter      | `png`, `serial`, `substack`, `social`, `newsletter` | `SubstackNewsletterPngSerializationToolchain` |  v2.0   |
| **Social Media (Video)**   | Serialization for TikTok                   | `mp4`, `serial`, `tiktok`, `social`                 | `TiktokMpegSerializationToolchain`            |  v1.0   |
|                            | Serialization for Youtube Shorts           | `mp4`, `serial`, `youtube`, `social`                | `YoutubeShortsMpegSerializationToolchain`     |  v1.0   |
| **Audio**                  | Serialization for Apple Podcast            | `m4a`, `serial`, `apple`, `social`, `podcast`       | `AaplPodcastM4ASerializationToolchain`        |  v1.0   |
|                            | Serialization for Google Podcast           | `m4a`, `serial`, `google`, `social`, `podcast`      | `GoogPodcastM4ASerializationToolchain`        |  v1.0   |
|                            | Serialization for Spotify Podcast          | `m4a`, `serial`, `spotify`, `social`, `podcast`     | `SpotifyPodcastM4ASerializationToolchain`     |  v1.0   |
|                            | Serialization for Soundcloud Podcast       | `m4a`, `serial`, `soundcloud`, `social`, `podcast`  | `SoundcloudPodcastM4ASerializationToolchain`  |  v1.0   |
|                            | Audible Full Manuscript Narrated           | `audible`, `full`, `narrated`                       | `AudibleFullNarratedManuscriptToolchain`      |  v1.0   |
|                            | Audible Full Manuscript Cast Read          | `audible`, `full`, `cast`                           | `AudibleFullCastReadManuscriptToolchain`      |  v1.0   |
|                            | Audible Sampler Manuscript Narrated        | `audible`, `sampler`, `narrated`                    | `AudibleSamplerNarratedManuscriptToolchain`   |  v1.0   |
|                            | Audible Sampler Manuscript Cast Read       | `audible`, `sampler`, `cast`                        | `AudibleSamplerCastReadManuscriptToolchain`   |  v1.0   |
| **Ebook**                  | PDF Ebook Full Manuscript                  | `ebook`, `pdf`, `full`                              | `EbookPdfFullManuscriptToolchain`             |  v2.0   |
|                            | PDF Ebook Sampler                          | `ebook`, `pdf`, `sampler`                           | `EbookPdfSamplerManuscriptToolchain`          |  v2.0   |
| **Print & Publishing**     | PDF Print Full Manuscript                  | `print`, `pdf`, `full`                              | `PrintPdfFullManuscriptToolchain`             |  v2.0   |
|                            | PDF Print Sampler Manuscript               | `print`, `pdf`, `sampler`                           | `PrintPdfSamplerManuscriptToolchain`          |  v2.0   |
|                            | InDesign Project                           | `indesign`, `project`, `full`                       | `InDesignProjectToolchain`                    |  v1.0   |
| **Production**             | Final Draft Project                        | `finaldraft`, `project`                             | `FinalDraftProjectToolchain`                  |  v1.0   |
|                            | Open Screenplay Format Project             | `osf`, `project`                                    | `OpenScreenplayFormatProjectToolchain`        |  v1.0   |
|                            | Scrivener Project                          | `scrivener`, `project`                              | `ScrivenerProjectToolchain`                   |  v1.0   |
|                            | Adobe Audition Project                     | `audition`, `project`                               | `AuditionProjectToolchain`                    |  v1.0   |
|                            | Arbortext Advanced Print Publisher Project | `arbortext`, `project`                              | `ArbortextProjectToolchain`                   |  v1.0   |
|                            | Table Read Script by Character             | `tableread`, `script`, `actor`                      | `TableReadScriptByCharacterToolchain`         |  v1.0   |
|                            | Table Read Script by Scene                 | `tableread`, `script`, `scene`                      | `TableReadScriptBySceneToolchain`             |  v1.0   |
|                            | Cast Callsheet                             | `callsheet`, `script`                               | `CastCallsheetToolchain`                      |  v1.0   |
|                            | Production Schedule                        | `production`, `schedule`                            | `ProductionScheduleToolchain`                 |  v1.0   |
|                            | Shooting Script                            | `shooting`, `script`                                | `ShootingScriptToolchain`                     |  v1.0   |
|                            | Shooting Script by Scene                   | `shooting`, `script`, `scene`                       | `ShootingScriptBySceneToolchain`              |  v1.0   |
|                            | Location Breakdown                         | `location`, `schedule`                              | `LocationBreakdownToolchain`                  |  v1.0   |
|                            | Shot List                                  | `shotlist`, `schedule`                              | `ShotListToolchain`                           |  v1.0   |
|                            | Film Script                                | `film`, `script`                                    | `FilmScriptToolchain`                         |  v1.0   |
|                            | Stage Play Script                          | `stage`, `drama`, `script`                          | `StagePlayScriptToolchain`                    |  v1.0   |
|                            | BBC Radio Drama Script                     | `bbc`, `radio`, `drama`, `script`                   | `BBCRadioDramaScriptToolchain`                |  v1.0   |
|                            | Storyboard                                 | `storyboard`, `visual`, `script`                    | `StoryboardToolchain`                         |  v1.0   |
|                            | Previz                                     | `previz`, `visual`, `script`                        | `PrevizToolchain`                             |  v1.0   |
|                            | Animatic                                   | `animatic`, `visual`, `script`                      | `AnimaticToolchain`                           |  v1.0   |
|                            | Unreal Engine Animatic                     | `unreal`, `visual`, `script`                        | `UnrealEngineAnimaticToolchain`               |  v1.0   |
|                            | Blackbox Video Render                      | `blackbox`,                                         | `BlackBoxVideoTestToolchain`                  |  v1.0   |
| **HTML**                   | AO3 by chapter                             | `ao3`, `html`, `chapter`                            | `Ao3HtmlChapterToolchain`                     |  v2.0   |
|                            | AO3 by scene                               | `ao3`, `html`, `scene`                              | `Ao3HtmlSceneToolchain`                       |  v2.0   |
|                            | Website Full Manuscript                    | `website`, `html`, `full`                           | `WebsiteHtmlManuscriptToolchain`              |  v2.0   |
|                            | Website by chapter                         | `website`, `html`, `chapter`                        | `WebsiteHtmlChapterToolchain`                 |  v2.0   |
|                            | Website by scene                           | `website`, `html`, `scene`                          | `WebsiteHtmlSceneToolchain`                   |  v2.0   |
|                            | Website serial                             | `website`, `html`, `serial`                         | `WebsiteHtmlSerialToolchain`                  |  v2.0   |