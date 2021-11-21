# findings
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//
//  Title : ani-cli Logical Evaluation
//  Author: @crux161 <crux161@protonmail.com>
//      with *Special Thanks* to github: @pystardust <pystardust@protonmail.com>
//
//  This document is licensed under the GNU General Public License v3.0 
//  The full text of which is available at <https://raw.githubusercontent.com/crux161/ani-cli/master/LICENSE>
//
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//  
//    Description: 
//     A evaluation of the logic, code, and presentation of the findings therein relevant to ani-cli scrapper script. 
//     This document serves as a record of the functions of ani-cli, and attempts to evaluate their logic for other 
//     gogoanime.me scrapers using example output captured live from the individual commands of the script. 
//     Personally, I would like to humbly thank github: @Pystardust <pystardust@protonmail.com> -- I forked the code 
//     for this paper from them and so would like to credit them here. This was a fun way to spend a lazy Sunday. ^__^;
//
//    Note on provided data:
//     The additional files provided here show the output of portions of different functions so as to model, and analyze the data 
//     in the example step by step. 
//
//    Proposed functionality:
//     Script asks for query and attempts to contact server, and scrape results from response page.    
//
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//
//    Variables of note:
//
//    base_url            : "https://gogoanime.cm"
//    anime_id            : internal site ID for anime. (url-safe format)
//    dub_prefix          : adds "-dub" suffix to anime_id for dub vs sub.
//    player_fn           : mpv (media player that accepts a url)
//    search              : search keyword (user input)
//    ep_no               : episode number (fmt: Int value)
//    embedded_video_url  : video playlist url 
//    video_url           : actual video file base_url
//    available_qualities : quality of file (360p/480p/720p/1080p etc)
//    is_download         : flag to set for download instead of stream
//    search_results      : formed by passing $query to search_anime()
//    selection_id        : url-safe string representing selected series name
//
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//
//    Listed functions and analysis. 
//
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//
// --help_text(): 
//      Prints help text.
//
// ======================================================================================================================================
//
// --die():
//      Exits because of bad state.
//
// ======================================================================================================================================
//
// --err(): 
//      Call when in error state. This is default? I'm not competent with shell scripting. 
//
//  ======================================================================================================================================
//
//  --search_anime(): 
//      Gets anime name, and its associated id.
//      This is given as a list of strings in a url-safe formatting.
//
//      Model Data:
//          $search = "key" (targetting "key-the-metal-idol")
//
//          +_1: curl -s "$base_url//search.html" -G -d "keyword=$search" |
//               sed -n -E 's_^[[:space:]]*<a href="/category/([^"]*)" title="([^"]*)".*_\1_p'
//
//          see findings/search_anime-stage1.out
//              findings/search_anime-stage2.out               
//
//  ======================================================================================================================================
//
//  --search_eps(): 
//      Gets available episodes for anime_id... 
//
//      Note: The sed command has been modified to work under macOS, most sed commands in the original script 
//            require this to function. Picky picky little details!
//
//   *Model Data: using id "key-the-metal-idol"
//       +_1: curl -s "$base_url/category/$anime_id" | 
//            sed -n -E '/^[[:space:]]*<a href="#" class="active" ep_start/s/.* '\''([0-9]*)'\'' ep_end = '\''([0-9]*)'\''.*/\2/p'
//            
//            Got result: 15
//
//            This command _should_ produce a int value for the total number of episodes based on the ID submitted. 
//
//           see findings/search_eps-stage1.out
//               findings/search_eps-stage2.out 
//
//  ======================================================================================================================================
//
//  --get_embedded_video_link(): 
//          Creates $embedded_video_url using $anime_id and $ep_no.
//
//     *Model Data: using id "key-the-metal-idol", and ep_no "1"
//      +_1: curl -s "$base_url/$anime_id${dub_prefix}-episode-$ep_no" |
//           sed -n -E '/^[[:space:]]*<a href="#" rel="100"/s/.*data-video="([^"]*)".*/https:\1/p'
//
//           Got result:
//              https://gogoplay1.com/embedplus?id=MTE2ODgx&token=Zb80uZt3zYw39-hooNNSaA&expires=1637531026
//
//           see findings/get_embedded_video_link-stage1.out
//               findings/get_embedded_video_link-stage2.out
//
//  ======================================================================================================================================             
//
//  --get_video_quality():
//         Gets video quality of $embedded_video_url
//      
//     Model Data: 
//        using $embedded_video_url as "https://gogoplay1.com/embedplus?id=MTE2ODgx&token=Zb80uZt3zYw39-hooNNSaA&expires=1637531026"
//
//  ====================================================================================================================================== 
//
//  --get_links():
//        Gets playlist data (m3u8) after parsing response from $embedded_video_url.
//        Model Data is submitted with `curl -s $embedded_video_url` and the is parsed 
//        by sed. The result is the value $video_url.
// 
//        macOS caveat:
//        However, in macOS sed does not play well with the given format. 
//        The modified sed command works in macOS to produce a proper response manually.
//        Hopefully, these can be passed ripgrep for a rust port. 
//
//    Model Data: using "https://gogoplay1.com/embedplus?id=MTE2ODgx&token=Zb80uZt3zYw39-hooNNSaA&expires=1637531026"
//
//        +_n: will try to produce video_url from embedded_video_url passed in under "$1"
//
//        +_1: curl -s "$embedded_video_url" |
//              sed -n -E '/^[[:space:]]*sources:/s/.*(https[^'\'']*).*/\1/p'
//
//              got result:
//                https://loadfast1.com/www02/1daa8558c97784d791d80d2c92ed2dfc/ep.1.1630752155.m3u8
//                https://loadfast1.com/www02/1daa8558c97784d791d80d2c92ed2dfc/ep.1.1630752155.m3u8
//
//        +_1: produces internal value "video_url"   
//
//          see findings/get_links-stage1.out 
//              findings/get_links-stage2.out
//
//  ====================================================================================================================================== 
//
// --dep_ch():
//    This is short for "Dependency Check"? -- this function calls die() if:
//        programs in $dep cannot be found using: command -v "$dep"
//
//        Note: dependencies are grep, sed, curl, video_player (mpv/vlc/etc),
//         Consider replacing these for rust port:
//
//             grep          ->   ripgrep
//             sed           ->   sd 
//             curl          ->   rust-curl 
//             video_player  ->   glide / libmpv 
//
// ======================================================================================================================================
//
// --get_search_query():
//    sets $query, used later to obtain $anime_id
//
// ======================================================================================================================================
//
// --anime_selection():
//    sets $anime_id using $search_results and collects $choice 
//    checks if $choice is valid
//    calls search_eps passing "selection_id" which is $anime_id
//
// ======================================================================================================================================
//
// --episode_selection():
//    sets $ep_choice_start and $ep_choice_end from user input 
//   
// ======================================================================================================================================
// 
// --check_input()
//    checks input for $ep_choice_start (delim)...
//
// ======================================================================================================================================
//
// --append_history():
//   writes $selection_id to $logfile path
//
// ======================================================================================================================================
//
// --open_selection()
//    iterates over $episodes calls open_episode passing $selection_id and $ep
//    creates $episode from: ${ep_choice_end :- ep_choice_start}
//
// ======================================================================================================================================
//
// --open_episode():
//    Descripttion of logic:
//        Function gets passed $anime_id as $selection_id, and episode $ep (episode can be a range?)
//
//        00_   | clears screen
//        01_   | checks if episode it out of range
//        02_   | says "getting data" :P
//        03_   | creates $embedded_video_url from $anime_id and $ep_no
//        04_   | creates $video_url from $embedded_video_url using get_links()
//        05_   | --MISC-- (Idk why these instructions occur here, because I don't know that much about shell script. *smh* )
//        06_   | checks for is_download flag-- true: download, flase: stream
//        -- Branch --
//        06_a    stream: calls player (mpv) with: 
//        ----              --http-header-fields="Referer: $embedded_video_url" "$video_url" >/dev/null 2>&1
//        ----                  
//        06_b    download: calls ffmpeg with: 
//        ----              -headers "Referer: $embedded_video_url" -i "$video_url" -c copy "${anime_id}-${episode}.mkv" >/dev/null 2>&1
//        END 
//
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
///                                    Thanks for reading, now port! Port my pretties!! Aa-ha-ha-ha!                                    ////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
