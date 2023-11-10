<?php
if (!function_exists('get_club_form')) {
    function get_club_form($atts)
    {
        extract(shortcode_atts(array(
            'club_id' => '',
            'date_to' => '',
            'classes' => '',
        ), $atts));

        $options = [
            'filter_by_clubs' => $club_id,
            'type'            => 'result',
            'sort_by_date'    => 'desc',
            'limit'           => 5,
        ];

        if ($date_to) {
            $date_to = explode(' ', $date_to)[0];

            if ('0000-00-00' !== $date_to && anwp_football_leagues()->helper->validate_date($date_to, 'Y-m-d')) {
                $options['date_to'] = DateTime::createFromFormat('Y-m-d', $date_to)->modify('-1 day')->format('Y-m-d');
            }
        }

        // Get latest matches
        $games = anwp_football_leagues()->competition->tmpl_get_competition_matches_extended($options);

        if (!empty($games) && is_array($games)) {
            $games = array_reverse($games);
        } else {
            return '';
        }

        // Mapping outcome labels
        $series_map = anwp_football_leagues()->data->get_series();

        ob_start();
?>
        <div class="club-form d-flex align-items-center justify-content-center <?php echo esc_attr($classes); ?>">
            <?php
            foreach ($games as $game) :
                if ((absint($club_id) === absint($game->home_club) && $game->home_goals > $game->away_goals) || (absint($club_id) === absint($game->away_club) && $game->away_goals > $game->home_goals)) {
                    $outcome_label = $series_map['w'];
                    $outcome_class = 'anwp-bg-success';
                } elseif ((absint($club_id) === absint($game->home_club) && $game->home_goals < $game->away_goals) || (absint($club_id) === absint($game->away_club) && $game->away_goals < $game->home_goals)) {
                    $outcome_label = $series_map['l'];
                    $outcome_class = 'anwp-bg-danger';
                } else {
                    $outcome_label = $series_map['d'];
                    $outcome_class = 'anwp-bg-warning';
                }
            ?>
                <span data-anwp-fl-match-tooltip data-match-id="<?php echo absint($game->match_id); ?>" class="d-inline-block club-form__item-pro anwp-text-white anwp-text-uppercase anwp-text-monospace <?php echo esc_attr($outcome_class); ?>">
                    <?php echo esc_html($outcome_label); ?>
                </span>
            <?php endforeach; ?>
        </div>
        <?php
        return ob_get_clean();
    }
}

// Add the shortcode with attributes
add_shortcode('club_form', 'get_club_form');

function custom_prediction_shortcode($atts)
{
    // Get post meta data
    $data = $atts;
    $predictions = [
        'percent'    => get_post_meta($data['match_id'], '_anwpfl_prediction_percent', true),
        'comparison' => get_post_meta($data['match_id'], '_anwpfl_prediction_comparison', true),
    ];

    // Extract numeric values from percent and comparison arrays
    $home_percent = intval(str_replace('%', '', $predictions['percent']['home']));
    $away_percent = intval(str_replace('%', '', $predictions['percent']['away']));

    $home_total = floatval($predictions['comparison']['total']['home']);
    $away_total = floatval($predictions['comparison']['total']['away']);

    $color_home = "#0085ba";
    $color_away = "#dc3545";

    // Calculate the correct average values
    $home_avg = round((($home_percent + $home_total) / (($home_percent + $home_total) + $away_percent + $away_total)) * 100);
    $away_avg = round((($away_percent + $away_total) / (($home_percent + $home_total) + $away_percent + $away_total)) * 100);

    // Create the shortcode output
    ob_start();

    // Check if at least one side has a value
    if ($home_total || $away_total) {
    ?>
        <div class="anwp-fl-prediction-percent mt-2 mx-2">
            <div class="d-flex align-items-center">
                <div class="team-stats__bar" style="width: <?php echo esc_attr($home_avg); ?>%; background-color: <?php echo esc_attr($color_home); ?>">
                    <span class="d-block anwp-text-center anwp-text-xs"><?php echo esc_html($home_avg); ?>%</span>
                </div>
                <div class="team-stats__bar" style="width: <?php echo esc_attr($away_avg); ?>%; background-color: <?php echo esc_attr($color_away); ?>">
                    <span class="d-block anwp-text-center anwp-text-xs"><?php echo esc_html($away_avg); ?>%</span>
                </div>
            </div>
        </div>
<?php
    }

    return ob_get_clean();
}

add_shortcode('custom_prediction', 'custom_prediction_shortcode');


function custom_odds_shortcode($atts)
{
    // Extract attributes and set defaults
    extract(shortcode_atts(array(
        'main_stage_id' => 0,
        'competition_id' => 0,
        'match_id' => 0,
        'book_id' => 4,
    ), $atts));

    $odds_competition = absint($main_stage_id) ?: absint($competition_id);
    $odds_data = get_post_meta($odds_competition, '_anwpfl_league_odds', true);

    if (empty($odds_data[$match_id])) {
        return '';
    }

    $odds_data = $odds_data[$match_id];

    if (empty($odds_data['odds'])) {
        return '';
    }

    $odds_clickable = AnWPFL_Premium_Options::get_value('import_api_api_ods_clickable');
    $bookmakers = anwp_football_leagues_premium()->match->get_bookmakers();
    $aff_link = AnWPFL_Premium_Options::get_value('import_api_bookmaker_' . absint($book_id));
    $load_alt = apply_filters('anwpfl/bookmaker/alternative_affiliate_link', '', $aff_link, $bookmakers[$book_id]);

    $desiredArray = $odds_data['odds'][1];
    $resultArray = $desiredArray['bookmakers'][$book_id];

    // Get the logo details
    $logoSrc = AnWP_Football_Leagues_Premium::url('public/img/bk/' . $bookmakers[$book_id]['img']);
    $logoAlt = $bookmakers[$book_id]['img'];

    // Generate the HTML
    // $output = '<img loading="lazy" src="' . esc_url($logoSrc) . '" class="anwp-w-120 anwp-object-contain anwp-h-20" alt="' . esc_attr($logoAlt) . '">';
    $output .= '<div class="d-flex justify-content-between mt-4 mx-2">';
    // Loop through the result array to generate links for options
    foreach ($resultArray as $option) {
        $optionName = $option['value'];
        $optionOdd = $option['odd'];

        $output .= '<a href="' . esc_url(do_shortcode($aff_link)) . '" target="_blank" class="anwp-link-without-effects odd-link px-3">';
        $output .= '<label>' . $optionName . '</label><span>' . $optionOdd . '</span>';
        $output .= '</a>';
    }
    $output .= '</div>';
    return $output;
}

// Register the shortcode
add_shortcode('custom_odds', 'custom_odds_shortcode');


function best_odds_shortcode($atts)
{
    // Extract attributes and set defaults
    extract(shortcode_atts(array(
        'main_stage_id' => 0,
        'competition_id' => 0,
        'match_id' => 0,
    ), $atts));

    $odds_competition = absint($main_stage_id) ?: absint($competition_id);
    $odds_data = get_post_meta($odds_competition, '_anwpfl_league_odds', true);

    if (empty($odds_data[$match_id])) {
        return '';
    }

    $odds_data = $odds_data[$match_id];

    if (empty($odds_data['odds'])) {
        return '';
    }

    $bestOdds = array(
        'Home' => 0,
        'Draw' => 0,
        'Away' => 0,
    );

    $bestBookmakers = array(
        'Home' => null,
        'Draw' => null,
        'Away' => null,
    );

    $desiredArray = $odds_data['odds'][1]['bookmakers'];

    // Loop through bookmakers and their odds
    foreach ($desiredArray as $bookmakerId => $bookmakerData) {
        foreach ($bookmakerData as $option) {
            $optionOdd = $option['odd'];
            $optionName = $option['value'];

            if ($optionOdd > $bestOdds[$optionName]) {
                $bestOdds[$optionName] = $optionOdd;
                $bestBookmakers[$optionName] = $bookmakerId;
            }
        }
    }

    $output = '';

    $bookmakers = anwp_football_leagues_premium()->match->get_bookmakers();
    $output .= '<div class="d-flex justify-content-between mt-4 mx-2">';

    foreach ($bestOdds as $optionName => $bestOdd) {
        $bookmakerId = $bestBookmakers[$optionName];
        if ($bookmakerId !== null) {
            $bookmakerData = $bookmakers[$bookmakerId];
            $aff_link = AnWPFL_Premium_Options::get_value('import_api_bookmaker_' . absint($bookmakerId));
            $logoSrc = AnWP_Football_Leagues_Premium::url('public/img/bk/' . $bookmakerData['img']);
            $logoAlt = $bookmakerData['img'];

            $output .= '<div class="anwp-text-sm">';
            $output .= '<a href="' . esc_url(do_shortcode($aff_link)) . '" target="_blank" class="odd-link p-2">';
            // $output .= '<img loading="lazy" src="' . esc_url($logoSrc) . '" class="anwp-w-120 anwp-object-contain anwp-h-20" alt="' . esc_attr($logoAlt) . '">';           
            $output .= '<span>' . $optionName . ':</span>';
            $output .= '<span class="odd-value">' . $bestOdd . '</span>';
            $output .= '</a>';
            $output .= '</div>';
        }
    }
    $output .= '</div>';

    return $output;
}

// Register the shortcode
add_shortcode('best_odds', 'best_odds_shortcode');

function total_stats_shortcode($atts)
{
    // Парсим атрибуты шорткода
    $atts = shortcode_atts(array(
        'home_club_id' => 0,
        'away_club_id' => 0,
        'season_id' => 0,
        'referee_id' => 0, // Добавляем атрибут referee_id
    ), $atts);

    // Проверяем, что указаны оба идентификатора команд, сезон и referee_id
    if (empty($atts['home_club_id']) || empty($atts['away_club_id']) || empty($atts['season_id'])) {
        return "Не указаны идентификаторы обеих команд, сезон и/или рефери.";
    }

    // Создаем аргументы для функции get_club_stats для команд
    $args_home = array(
        'club_id' => $atts['home_club_id'],
        'stats' => ['goals', 'goals_conceded', 'cards_y', 'corners'],
        'season_id' => $atts['season_id'],
        'limit' => 5, // Добавляем ограничение на последние 5 матчей
    );

    $args_away = array(
        'club_id' => $atts['away_club_id'],
        'stats' => ['goals', 'goals_conceded', 'cards_y', 'corners'],
        'season_id' => $atts['season_id'],
        'limit' => 5, // Добавляем ограничение на последние 5 матчей
    );

    // Создаем аргументы для функции get_referee_games
    $args_referee = (object) wp_parse_args(
        [
            'referee_id' => $atts['referee_id'],
            'competition_id' => '',
            'season_id' => $atts['season_id'],
            'league_id' => '',
            'show_secondary' => 0,
            'per_game' => 1,
            'block_width' => 1,
            'stats' => ['card_y'],
            'class' => '',
            'header' => '',
            'layout' => '',
            'notes' => '',
            'show_games' => 1,
            'type' => 'result',
        ]
    );

    // Вызываем функции get_club_stats и get_referee_games для получения статистики
    $home_stats = anwp_football_leagues_premium()->club->get_club_stats($args_home);
    $away_stats = anwp_football_leagues_premium()->club->get_club_stats($args_away);
    $referee_stats = anwp_football_leagues()->referee->get_referee_games($args_referee, 'stats');

    // Проверяем, есть ли данные о голах и пропущенных голах для обеих команд и рефери
    if (empty($home_stats) || empty($away_stats)) {
        return "Недостаточно данных о голах и/или пропущенных голах для указанных команд и рефери.";
    }

    // Извлекаем статистику для голов, пропущенных голов, желтых карточек и угловых
    $home_goals_scored = $home_stats['goals']['h'];
    $home_goals_conceded = $home_stats['goals_conceded']['h'];
    $home_yellow_cards = $home_stats['cards_y']['h'];
    $home_corners = $home_stats['corners']['h'];

    $away_goals_scored = $away_stats['goals']['a'];
    $away_goals_conceded = $away_stats['goals_conceded']['a'];
    $away_yellow_cards = $away_stats['cards_y']['a'];
    $away_corners = $away_stats['corners']['a'];

    // Расчет среднего количества голов, желтых карточек и угловых за матч
    $homeGoalsPerMatch = $home_goals_scored / $home_stats['played']['h'];
    $homeGoalsConcededPerMatch = $home_goals_conceded / $home_stats['played']['h'];
    $homeYellowCardsPerMatch = $home_yellow_cards / $home_stats['played']['h'];
    $homeCornersPerMatch = $home_corners / $home_stats['played']['h'];

    $awayGoalsPerMatch = $away_goals_scored / $away_stats['played']['a'];
    $awayGoalsConcededPerMatch = $away_goals_conceded / $away_stats['played']['a'];
    $awayYellowCardsPerMatch = $away_yellow_cards / $away_stats['played']['a'];
    $awayCornersPerMatch = $away_corners / $away_stats['played']['a'];

    // Учитываем разницу между средними значениями обеих команд и прибавляем их друг к другу
    $expectedHomeGoals = ($homeGoalsPerMatch + $awayGoalsConcededPerMatch) / 2;
    $expectedAwayGoals = ($awayGoalsPerMatch + $homeGoalsConcededPerMatch) / 2;
    $expectedYellowCards = $homeYellowCardsPerMatch + $awayYellowCardsPerMatch;

    // Если есть данные о рефери, учитываем среднее количество желтых карточек рефери
    if (!empty($referee_stats)) {
        $referee_yellow_cards = 0;
        foreach ($referee_stats as $referee_game) {
            if (isset($referee_game->home_cards_y)) {
                $referee_yellow_cards += absint($referee_game->home_cards_y);
            }
            if (isset($referee_game->away_cards_y)) {
                $referee_yellow_cards += absint($referee_game->away_cards_y);
            }
        }
        $average_referee_yellow_cards = (count($referee_stats) > 0) ? $referee_yellow_cards / count($referee_stats) : 0;
        $expectedYellowCards = ($expectedYellowCards + $average_referee_yellow_cards) / 2;
    }

// Формируем ответ
$goalsValue = round($expectedHomeGoals + $expectedAwayGoals, 1);
$yellowCardsValue = round($expectedYellowCards, 1);
$cornersValue = round($homeCornersPerMatch + $awayCornersPerMatch, 1);

$forecast = "
<div class='anwp-row'>
    <div class='anwp-fl-builder-block anwp-col-md-6 anwp-col-xl-4'>
        <div class='grid-stat goals-grid rh-cartbox rh-flex-center-align'>
            <svg class='icon__ball anwp-icon--stats-goal p-0 mr-2' style='fill:#fff;'><use xlink:href='#icon-ball'></use></svg>
            <span class='mr-2 font120 fontbold'>" . esc_html($goalsValue) . __("Goals / Match", 'wizodds') . "</span>
        </div>
    </div>
    <div class='anwp-fl-builder-block anwp-col-md-6 anwp-col-xl-4'>
        <div class='grid-stat cards-grid rh-cartbox rh-flex-center-align'>
            <svg class='icon__ball mr-2 anwp-flex-none mr-2'><use xlink:href='#icon-card_y'></use></svg>
            <span class='mr-2 font120 fontbold'>" . esc_html($yellowCardsValue) . __("Cards / Match", 'wizodds') . "</span>
        </div>
    </div>
    <div class='anwp-fl-builder-block anwp-col-md-6 anwp-col-xl-4'>
        <div class='grid-stat corners-grid rh-cartbox rh-flex-center-align'>
            <img src='https://wizodds.com/wp-content/uploads/corner.svg' style='width: 20px;' class='mr-2' />
            <span class='mr-2 font120 fontbold'>" . esc_html($cornersValue) . __("Corners / Match", 'wizodds') . "</span>
        </div>
    </div>
</div>
";

return $forecast;


}

// Регистрируем шорткод
add_shortcode('total_stats', 'total_stats_shortcode');


function poissonProbability($lambda, $k)
{
    $e = exp(-$lambda);
    $probability = pow($lambda, $k) * $e / factorial($k);

    return $probability;
}

function factorial($n)
{
    if ($n <= 1) {
        return 1;
    }
    return $n * factorial($n - 1);
}

function goals_probability_shortcode($atts)
{
    $atts = shortcode_atts(array(
        'home_club_id' => 0, // Идентификатор домашней команды
        'away_club_id' => 0, // Идентификатор гостевой команды
        'greater_than' => 2.5, // Порог для вероятности
    ), $atts);

    $home_club_id = (int) $atts['home_club_id'];
    $away_club_id = (int) $atts['away_club_id'];
    $greater_than = (float) $atts['greater_than'];

    if ($home_club_id <= 0 || $away_club_id <= 0 || $greater_than <= 0) {
        return "Неверные параметры для расчета вероятности.";
    }

    // Создаем аргументы для функции get_club_stats
    $args_home = array(
        'club_id' => $home_club_id,
        'stats' => ['goals', 'goals_conceded'],
    );

    $args_away = array(
        'club_id' => $away_club_id,
        'stats' => ['goals', 'goals_conceded'],
    );

    // Вызываем функцию get_club_stats для получения статистики
    $home_stats = anwp_football_leagues_premium()->club->get_club_stats($args_home);
    $away_stats = anwp_football_leagues_premium()->club->get_club_stats($args_away);

    // Проверяем, есть ли данные о голах и пропущенных голах для обеих команд
    if (empty($home_stats) || empty($away_stats)) {
        return "Недостаточно данных о голах и/или пропущенных голах для указанных команд.";
    }

    // Извлекаем статистику для голов и пропущенных голов
    $home_goals_scored = $home_stats['goals']['h'];
    $home_goals_conceded = $home_stats['goals_conceded']['h'];
    $away_goals_scored = $away_stats['goals']['a'];
    $away_goals_conceded = $away_stats['goals_conceded']['a'];

    // Расчет среднего количества голов, забитых и пропущенных командами
    $homeGoalsPerMatch = $home_goals_scored / $home_stats['played']['h'];
    $awayGoalsPerMatch = $away_goals_scored / $away_stats['played']['a'];
    $homeGoalsConcededPerMatch = $home_goals_conceded / $home_stats['played']['h'];
    $awayGoalsConcededPerMatch = $away_goals_conceded / $away_stats['played']['a'];

    // Учитываем разницу между средними голами, забитыми одной командой, и средними голами, пропущенными другой командой
    $expectedHomeGoals = ($homeGoalsPerMatch + $awayGoalsConcededPerMatch) / 2;
    $expectedAwayGoals = ($awayGoalsPerMatch + $homeGoalsConcededPerMatch) / 2;

    // Прогнозируем общее количество голов
    $totalGoals = $expectedHomeGoals + $expectedAwayGoals;

    // Calculate probability and cap it at 100%
    $probability = 1 - array_sum(array_map(function ($k) use ($totalGoals) {
        return poissonProbability($totalGoals, $k);
    }, range(0, floor($greater_than))));

    $probability = min($probability, 1); // Cap at 100%

    // Determine the background color based on the probability value
    if ($probability <= 0.4) {
        $backgroundColor = 'red';
    } elseif ($probability <= 0.6) {
        $backgroundColor = 'orange';
    } else {
        $backgroundColor = 'green';
    }

    // Generate the HTML with the calculated background color
    $html = "<div class='grid-stat rh-cartbox' style='border-width: 3px;border-color: $backgroundColor;'>
    <span class='mr-2 font120 fontbold'>" . esc_html(round($probability * 100)) . "%" . "</span>
    " . sprintf(__('Over %s', 'wizodds'), esc_html($greater_than)) . "
</div>";

    return $html;
}

add_shortcode('goals_probability', 'goals_probability_shortcode');

function both_teams_to_score_probability_shortcode($atts)
{
    $atts = shortcode_atts(array(
        'home_club_id' => 0,
        'away_club_id' => 0,
        'match_id' => 0,
        'probability_threshold' => 70, // Порог вероятности
    ), $atts);

    $home_club_id = (int) $atts['home_club_id'];
    $away_club_id = (int) $atts['away_club_id'];
    $match_id = (int) $atts['match_id'];
    $probability_threshold = (float) $atts['probability_threshold'];

    if ($home_club_id <= 0 || $away_club_id <= 0 || $match_id <= 0 || $probability_threshold <= 0) {
        return "Неверные параметры для расчета вероятности.";
    }

    // Создаем аргументы для функции get_club_stats
    $args_home = array(
        'club_id' => $home_club_id,
        'stats' => ['goals', 'clean_sheets'],
    );

    $args_away = array(
        'club_id' => $away_club_id,
        'stats' => ['goals', 'clean_sheets'],
    );

    // Вызываем функцию get_club_stats для получения статистики
    $home_stats = anwp_football_leagues_premium()->club->get_club_stats($args_home);
    $away_stats = anwp_football_leagues_premium()->club->get_club_stats($args_away);

    // Получаем данные из прогноза
    $predictions = [
        'percent'    => get_post_meta($match_id, '_anwpfl_prediction_percent', true),
        'comparison' => get_post_meta($match_id, '_anwpfl_prediction_comparison', true),
    ];

    // Проверяем, есть ли данные о голах и "Clean Sheets" для обеих команд
    if (empty($home_stats) || empty($away_stats) || empty($predictions['percent']) || empty($predictions['comparison']['goals'])) {
        return "";
    }

    // Извлекаем статистику для голов и "Clean Sheets" команд
    $home_goals_scored = $home_stats['goals']['h'];
    $away_goals_scored = $away_stats['goals']['a'];
    $home_clean_sheets = $home_stats['clean_sheets']['h'];
    $away_clean_sheets = $away_stats['clean_sheets']['a'];

    // Рассчитываем среднее количество голов, забитых каждой командой
    $homeGoalsPerMatch = $home_stats['played']['h'] > 0 ? $home_goals_scored / $home_stats['played']['h'] : 0;
    $awayGoalsPerMatch = $away_stats['played']['a'] > 0 ? $away_goals_scored / $away_stats['played']['a'] : 0;

    // Рассчитываем вероятность "Обе команды забьют" с использованием модифицированного распределения Пуассона
    $lambda_home = max(0.1, $homeGoalsPerMatch - $home_clean_sheets / $home_stats['played']['h']);
    $lambda_away = max(0.1, $awayGoalsPerMatch - $away_clean_sheets / $away_stats['played']['a']);

    // Вероятность что хотя бы 1 гол забьют
    $probability_both_teams_to_score = 1 - poissonProbability($lambda_home, 0) * poissonProbability($lambda_away, 0);

    // Уменьшаем порог вероятности пропорционально
    $probability_both_teams_to_score *= ($probability_threshold / 100);

    // Extract numeric values from percent and comparison arrays
    $home_percent = intval(str_replace('%', '', $predictions['percent']['home']));
    $draw_percent = intval(str_replace('%', '', $predictions['percent']['draw']));
    $away_percent = intval(str_replace('%', '', $predictions['percent']['away']));

    $home_total = floatval($predictions['comparison']['goals']['home']);
    $away_total = floatval($predictions['comparison']['goals']['away']);

    // Calculate the correct average values
    $home_avg = ($home_percent + $home_total) / 2;
    $draw_avg = ($draw_percent + 0) / 2;
    $away_avg = ($away_percent + $away_total) / 2;

    // Объединяем вероятности
    $combined_probability = ($home_avg + $draw_avg + $away_avg) / 3;

    // Итоговая вероятность
    $final_probability = min($combined_probability, $probability_both_teams_to_score);

    // Determine the border color based on the final probability value
    if ($final_probability <= 0.4) {
        $borderColor = 'red';
    } elseif ($final_probability <= 0.6) {
        $borderColor = 'orange';
    } else {
        $borderColor = 'green';
    }

    $html = "<div class='grid-stat rh-cartbox' style='border-width: 3px;border-color: $borderColor;'><span class='mr-2 font120 fontbold'>" . round($final_probability * 100) . "%</span>BTTS</div>";

    return $html;
}


add_shortcode('both_teams_to_score_probability', 'both_teams_to_score_probability_shortcode');

function custom_stat_shortcode($atts) {
    $atts = shortcode_atts(array(
        'club_id' => 0,
        'season_id' => 0,
        'field' => '', // Define a field attribute
        'place' => 'h', // Default to home statistics
    ), $atts);

    $club_id = (int) $atts['club_id'];
    $season_id = (int) $atts['season_id'];
    $field = sanitize_key($atts['field']);
    $place = strtolower(sanitize_text_field($atts['place'])); // Convert to lowercase

    // Define an array of valid fields
    $valid_fields = ['cards_y', 'corners', 'goals_conceded', 'goals', 'wins', 'draws', 'losses', 'clean_sheets'];

    // Check if the provided field is valid
    if (!in_array($field, $valid_fields)) {
        return "Invalid field specified for custom_stat shortcode.";
    }

    // Check if the provided place is 'h' (home) or 'a' (away)
    if ($place !== 'h' && $place !== 'a') {
        return "Invalid place specified. Use 'h' for home or 'a' for away.";
    }

    // Create arguments for get_club_stats function
    $args = [
        'club_id' => $club_id,
        'season_id' => $season_id,
        'stats' => ['played', $field], // Include 'played' for total matches
        'limit' => 5, // Add any other desired options
    ];

    // Get club statistics using the get_club_stats function
    $club_stats = anwp_football_leagues_premium()->club->get_club_stats($args);

    // Access the specific statistic based on the place (home or away)
    $played = $club_stats['played'][$place];
    $statistic = $club_stats[$field][$place];

    // Calculate the statistic per match (not total) if it's not one of the excluded fields
    if (!in_array($field, ['wins', 'draws', 'losses', 'clean_sheets'])) {
        $statistic_per_match = $played > 0 ? $statistic / $played : 0;
        return round($statistic_per_match, 1); // Round to two decimal places
    } else {
        return $statistic; // Return the field value as is for the excluded fields
    }
}

add_shortcode('custom_stat', 'custom_stat_shortcode');


?>
