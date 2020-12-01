﻿# laravel-cashier-stripe-connect

<p>
    <a href='https://ko-fi.com/C1C510DUQ' target='_blank'>
	<img height='36' style='border:0px;height:36px;' src='https://az743702.vo.msecnd.net/cdn/kofi3.png?v=2' border='0' alt='Buy Me a Coffee at ko-fi.com' />
    </a>
</p>

> Adds Stripe Connect functionality to Laravel's main billing package, Cashier.

## Install

1. Enable Stripe Connect in your [dashboard settings](https://dashboard.stripe.com/settings).
2. Install Cashier: ``composer require laravel/cashier``
3. Install package: ``composer require expdev07/laravel-cashier-stripe-connect``.
4. Run migrations: ``php artisan migrate``.

**Note:** the package will not work as intended if you do not install [Laravel's official Cashier package](https://laravel.com/docs/8.x/billing) first.

## Use

The library builds on the official [Cashier](https://laravel.com/docs/8.x/billing) library, so getting up and started is a breeze.

### Setup model

Add the ``Billable`` traits to your model. You can use them individually or together. You can also create your own ``Billable`` trait and put them together there.

```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Cashier\Billable as CashierBillable;
use ExpDev07\CashierConnect\Billable as ConnectBillable;

class User extends Authenticatable
{
    use CashierBillable;
    use ConnectBillable;

    ///

}
```

### Create controller

Create a controller to manage on-boarding process. The example below registers an Express account for the user.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use URL;

class StripeController extends Controller
{

    /**
     * Creates an onboarding link and redirects the user there.
     *
     * @param Request $request
     * @return RedirectResponse
     */
    public function board(Request $request): RedirectResponse
    {
        return $this->handleBoardingRedirect($request->user());
    }

    /**
     * Handles returning from completing the onboarding process.
     *
     * @param Request $request
     * @return RedirectResponse
     */
    public function returning(Request $request): RedirectResponse
    {
        return $this->handleBoardingRedirect($request->user());
    }

    /**
     * Handles refreshing of onboarding process.
     *
     * @param Request $request
     * @return RedirectResponse
     */
    public function refresh(Request $request): RedirectResponse
    {
        return $this->handleBoardingRedirect($request->user());
    }

    /**
     * Handles the redirection logic of Stripe onboarding for the given user. Will create account and redirect user to
     * onboarding process or redirect to account dashboard if they have already completed the process.
     *
     * @param User $user
     * @return RedirectResponse
     */
    private function handleBoardingRedirect(User $user): RedirectResponse
    {
        // Redirect to dashboard if onboarding is already completed.
        if ($user->hasStripeAccountId() && $user->hasCompletedOnboarding()) {
            return $user->redirectToAccountDashboard();
        }

        // Delete account if already exists and create new express account with weekly payouts.
        $user->deleteAndCreateStripeAccount('express', [
            'settings' => [
                'payouts' => [ 'schedule' => [ 'interval' => 'weekly', 'weekly_anchor' => 'friday' ] ]
            ]
        ]);

        // Redirect to account onboarding otherwise.
        return $user->redirectToAccountLink('account_onboarding', [
            'return_url' => URL::to('/api/stripe/return?api_token=' . $user->api_token),
            'refresh_url' => URL::to('/api/stripe/refresh?api_token=' . $user->api_token),
        ]);
    }

}
```



## License

Please refer to [LICENSE.md](https://github.com/ExpDev07/laravel-cashier-stripe-connect/blob/master/LICENSE.md) for this project's license.

## Contributors

This list only contains some of the most notable contributors. For the full list, refer to [GitHub's contributors graph](https://github.com/ExpDev07/legacy-rp-admin-v3/graphs/contributors).
* ExpDev07 (Marius) - creator and maintainer.

## Thanks to

[Taylor Otwell](https://twitter.com/taylorotwell) for his amazing framework and [all the contributors of Cashier](https://github.com/laravel/cashier-stripe/graphs/contributors).
