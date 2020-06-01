# ue4-chest2d-sample

![UE4](https://img.shields.io/badge/UE4-4.25+-blue)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/Nauja/ue4-jetpack-sample/master/LICENSE)

Sample of a 2D chest done with Paper2D.

![Preview](https://github.com/Nauja/ue4-chest2d-sample/raw/master/docs/preview.gif)

This project is an example of how to write an interactable chest in a Paper2D game, with the constraint of
being fully replicated over network.

Features:
  * Pixel perfect camera/sprites
  * True sprite colors
  * Custom **Interact** action (E)
  * C++ RPC to replicate **Interact** action
  * C++ interactable chest

### Pixel perfect sprites

Sprites are coming from [The Spriters Resource](https://www.spriters-resource.com/).

As an example, here is the charset used for Mario:

![Mario](https://github.com/Nauja/ue4-chest2d-sample/raw/master/Content/Mario/Textures/T_Mario.png)

You can see that the size of one frame is really small (16x32 pixels). If it were displayed as is
in UE4, then all your physics would have to be configured with really small values and placing or moving
Actors in the editor would become complicated due to their small size.

So, in this sample all sprites are configured with a **pixels per unit** of **0.5** that effectively make them scaled
by two:

![PixelsPerUnit](https://github.com/Nauja/ue4-chest2d-sample/raw/master/docs/pixelperfect_sprite.png)

In **ASampleCharacter.cpp**, we configure the **SideViewCameraComponent** with an **OrthoWidth** of **1280** pixels:

```cpp
SideViewCameraComponent = CreateDefaultSubobject<UCameraComponent>(TEXT("SideViewCamera"));
SideViewCameraComponent->ProjectionMode = ECameraProjectionMode::Orthographic;
SideViewCameraComponent->OrthoWidth = 1280.0f;
SideViewCameraComponent->SetupAttachment(CameraBoom, USpringArmComponent::SocketName);
```

### True sprite colors

One common problem is to keep the colors of your sprites intact.
By default, many post processing effects are enabled and are causing sprites
not to be rendered correctly. In this sample, all post processing effects are disabled
with this single command:

```cpp
APlayerController* Controller = UGameplayStatics::GetPlayerController(GetWorld(), 0);
check(Controller);
Controller->ConsoleCommand(TEXT("showflag.postprocessing 0"));
```

Also, in **Project Settings > Engine > Rendering**, make sure to uncheck the **Mobile HDR** option and all post processing options such as **Bloom**, **Auto Exposure**, **Anti-Aliasing**, etc.

### C++ RPC to replicate Interact action

Pressing the **Interact** button client-size simply calls a RPC server-side to handle the action.

Here is the definition in **ASampleCharacter.h**:

```cpp
UFUNCTION()
void Interact();

UFUNCTION(reliable, Server, WithValidation)
void Server_Interact();
```

Here is the implementation in **ASampleCharacter.cpp**:

```cpp
void ASampleCharacter::Interact()
{
    if (GetLocalRole() < ROLE_Authority) {
        Server_Interact();
    }
    else
    {
        TSet<AActor*> Actors;
        GetOverlappingActors(Actors, TSubclassOf<ASampleInteractableActor>());
        for (auto Actor : Actors)
        { 
            static_cast<ASampleInteractableActor*>(Actor)->Interact(this);
        }
    }
}

bool ASampleCharacter::Server_Interact_Validate()
{
    return true;
}

void ASampleCharacter::Server_Interact_Implementation()
{
    Interact();
}
```

Please note that this RPC has to be called from an Actor owned by the client for it to works.
Per consequence it would not be possible to move it directly into the class of our
interactable Actor. Missing this point can give you a hard time trying to figure out why
the RPC is not called on server.

### Interactable chest

wip
